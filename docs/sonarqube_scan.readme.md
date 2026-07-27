# sonarqube_scan.yml

Reusable workflow that spins up an ephemeral SonarQube Community Build
instance, scans the calling repo for SAST findings and code-quality issues,
and fails the calling job if the Quality Gate doesn't pass.

Ephemeral by design: each run gets a brand-new, historyless SonarQube
instance that's destroyed at the end, rather than pointing at a persistent
server. No server to host, no `SONAR_TOKEN`/`SONAR_HOST_URL` secrets to
manage - but also no analysis history across runs, so the Quality Gate's
"new code" conditions evaluate against the whole codebase every run, not
just what changed since the last one. If you need real trend data or
new-code-only gating, point this workflow (or a fork of it) at a persistent
SonarQube Server instance instead.

## Usage

```yaml
code_scan:
  needs: test
  permissions:
    contents: read
  uses: rftecpro-ai/reusable-workflows/.github/workflows/sonarqube_scan.yml@v1
  with:
    project_key: my-project
    sources: .
    sonarqube_tag: "community"
    fail_on_quality_gate: true
```

## Inputs

| Name                    | Required | Default     | Description                                              |
| ------------------------ | -------- | ----------- | --------------------------------------------------------- |
| `project_key`             | yes      |             | Unique SonarQube project key for this scan                |
| `sources`                 | no       | `.`         | Path (relative to repo root) analyzed as the project base dir |
| `sonarqube_tag`           | no       | `community` | `sonarqube` Docker Hub image tag to run                   |
| `fail_on_quality_gate`    | no       | `true`      | Fail the job if the Quality Gate doesn't pass              |

## What it produces

- A job summary listing every open issue SonarQube found (severity, rule, message, location).
- A hard failure if `fail_on_quality_gate` is true and the Quality Gate status isn't `OK`.
- Nothing persists after the job ends - the SonarQube instance (and any issue history) is destroyed in the last step regardless of outcome. There is no Security tab integration and no cross-run trend; re-run the workflow to see current findings.

## Caller requirements

- Must grant `permissions: contents: read` on the calling job - checkout is
  the only thing this workflow's own steps need beyond what the ephemeral
  SonarQube instance does locally.
- No repository secrets are required. The instance generates its own
  short-lived analysis token internally and is torn down at the end of the
  job either way.
- The calling repo must allow this repo to be referenced as a reusable
  workflow (Settings → Actions → General → Access, on either this repo or
  the calling repo depending on org policy).
- Runs on `ubuntu-latest` (or an equivalent runner where the job can `sudo
  sysctl` and run Docker) - the `vm.max_map_count` step needs host-level
  access that a locked-down or containerized runner may not have.
