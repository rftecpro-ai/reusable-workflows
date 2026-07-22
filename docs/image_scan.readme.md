# image_scan.yml

Reusable workflow that builds a single-arch (amd64) local image from the calling
repo, scans it with [Trivy](https://github.com/aquasecurity/trivy), and fails the
calling job if any HIGH/CRITICAL vulnerability with an available fix is found.

## Usage

```yaml
image_scan:
  needs: test
  permissions:
    contents: read
    security-events: write # required to upload SARIF to the Security tab
  uses: rftecpro-ai/reusable-workflows/.github/workflows/image_scan.yml@v1
  with:
    dockerfile_context: .
    image_ref: scan-target
    version: ${{ needs.test.outputs.version }}
    trivy_version: "0.70.0"
    severity: "HIGH,CRITICAL"
```

## Inputs

| Name                 | Required | Default    | Description                                              |
| -------------------- | -------- | ---------- | --------------------------------------------------------- |
| `dockerfile_context`  | no       | `.`        | Build context passed to `docker/build-push-action`        |
| `image_ref`           | yes      |            | Image name to build and scan, without tag                 |
| `version`             | yes      |            | Tag applied to `image_ref`, and used in report/artifact names |
| `trivy_version`       | no       | `0.70.0`   | `aquasec/trivy` image tag to run                           |
| `severity`            | no       | `HIGH,CRITICAL` | Comma-separated severities to gate on                 |

## What it produces

- A SARIF upload to the calling repo's Security tab (unfiltered, includes unfixed CVEs, for tracking).
- A fenced Trivy table in the job summary (unfiltered, same reason).
- An HTML report and the SARIF file uploaded as a `trivy-report-<version>` artifact.
- A hard failure (`exit-code 1`) if any HIGH/CRITICAL CVE with a fix available is present. Unfixed CVEs never block the build, since there's nothing actionable to do about them yet.

## Caller requirements

- Must grant `permissions: security-events: write` on the calling job (this workflow's own `security-events: write` is a ceiling, not a grant).
- The calling repo must allow this repo to be referenced as a reusable workflow (Settings → Actions → General → Access, on either this repo or the calling repo depending on org policy).
