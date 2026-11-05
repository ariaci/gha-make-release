# gha-make-release

A reusable GitHub Composite Action for creating GitHub Releases with build artifacts and SHA256 checksums, compatible with both GitHub Actions and local executions using [`act`](https://github.com/nektos/act).

## Overview

This action automates the creation of GitHub Releases from previously downloaded workflow artifacts.

It performs the following steps:

- **Flatten artifacts:** Moves files from nested artifact directories into a single directory.
- **Generate checksums:** Creates a `SHA256SUMS` file containing SHA256 checksums for all release artifacts.
- **Create release:** Publishes the artifacts and checksums as a GitHub Release using [`softprops/action-gh-release`](https://github.com/softprops/action-gh-release).
- **Local `act` support:** Skips GitHub Release creation and uploads all prepared release artifacts, including `SHA256SUMS`, as a single workflow artifact with a configurable name.

The execution environment is detected automatically using the `ACT` environment variable.

## Usage

Download the required artifacts and invoke the action in your release job:

```yaml
release:
  name: Create Release
  runs-on: ubuntu-latest
  permissions:
    contents: write

  steps:
    - name: Download artifacts
      uses: actions/download-artifact@v8
      with:
        path: artifacts

    - name: Create GitHub Release
      uses: ariaci/gha-make-release@v1
      with:
        path: artifacts
```

The same workflow can be executed locally using `act` without creating a GitHub Release.

## Local Execution with `act`

When executed using [`act`](https://github.com/nektos/act), the action prepares the release artifacts exactly as it would in GitHub Actions, including directory flattening and SHA256 checksum generation.

Instead of publishing a GitHub Release, all prepared files are uploaded to the local `act` artifact server as a single workflow artifact.

By default, the artifact is named `act-release-artifacts`. This name can be customized using the optional `act-artifact-name` input:

```yaml
- name: Create GitHub Release
  uses: ariaci/gha-make-release@v1
  with:
    path: artifacts
    act-artifact-name: my-release-artifacts
```

The `act-artifact-name` input is only used during local `act` execution and has no effect on GitHub Releases.

This allows the complete release artifact set to be inspected and verified locally without publishing an actual release.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `path` | No | `artifacts` | Directory containing previously downloaded workflow artifacts. |
| `act-artifact-name` | No | `act-release-artifacts` | Name of the workflow artifact uploaded during local `act` execution. |

## Requirements

- GitHub Actions or a compatible `act` environment.
- Downloaded artifacts available in the specified directory.
- `contents: write` permission for creating GitHub Releases.
- Unique filenames across all artifact directories to avoid collisions during flattening.
- Linux runner with Bash and standard command-line utilities.
- Local artifact server support when executing with `act`.

The release is created using the current Git reference (`github.ref_name`) as its tag.

## License

This project is licensed under the [MIT License](LICENSE).
