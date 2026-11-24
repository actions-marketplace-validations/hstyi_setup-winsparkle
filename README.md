# Setup WinSparkle

A lightweight [GitHub composite action](https://docs.github.com/en/actions/tutorials/create-actions/create-a-composite-action) that downloads a specified [WinSparkle](https://winsparkle.org/) release and adds its command-line tools to `PATH`.

Designed for Windows build and release workflows. No Node.js runtime or external action dependencies are required.

## Usage

Add the following step to a workflow running on Windows:

```yaml
jobs:
  release:
    runs-on: windows-latest
    steps:
      - name: Set up WinSparkle
        uses: hstyisetup-winsparkle@v1
        with:
          version: '0.9.4'

      - name: Verify installation
        shell: pwsh
        run: Get-Command winsparkle-tool.exe
```

The `version` input is optional. When omitted, it defaults to `0.9.4`.

## Inputs

| Input | Description | Required | Default |
| --- | --- | --- | --- |
| `version` | WinSparkle release version, without the leading `v` | No | `0.9.4` |

## Outputs

| Output | Description |
| --- | --- |
| `bin-path` | Absolute path to the directory containing WinSparkle command-line tools |
| `install-path` | Absolute path to the extracted WinSparkle package directory |

You can use the outputs to locate the downloaded files:

```yaml
- name: Set up WinSparkle
  id: winsparkle
  uses: hstyisetup-winsparkle@v1

- name: Inspect WinSparkle files
  shell: pwsh
  env:
    WINSPARKLE_BIN: ${{ steps.winsparkle.outputs.bin-path }}
    WINSPARKLE_HOME: ${{ steps.winsparkle.outputs.install-path }}
  run: |
    Get-ChildItem -LiteralPath $env:WINSPARKLE_BIN
    Get-ChildItem -LiteralPath $env:WINSPARKLE_HOME
```

## Requirements

- A **Windows** GitHub Actions runner (for example, `windows-latest`).
- PowerShell 7 (`pwsh`), included on GitHub-hosted Windows runners.
- Network access to GitHub Releases to download the selected WinSparkle archive.

The installed `bin` directory is added to `PATH` for **subsequent steps in the same job**. It does not persist across jobs.

## How it works

1. Downloads `WinSparkle-<version>.zip` from the official [WinSparkle releases](https://github.com/vslavik/winsparkle/releases).
2. Extracts it into the runner's temporary directory.
3. Locates the release's `bin` directory and adds it to `GITHUB_PATH`.
4. Exposes the tool and installation paths as action outputs.

The action installs WinSparkle tools only; it does **not** link WinSparkle into your application or automatically sign updates. See the [WinSparkle documentation](https://winsparkle.org/guides/getting-started/) for integration and signing details.

## Development

The repository includes a smoke-test workflow at `.github/workflows/ci.yml`. It runs the action on `windows-latest` and verifies that `winsparkle-tool.exe` is available through `PATH`.

No build step is needed: the `action.yml` at the repository root is the published action.

## License

The code in this repository is licensed under the [MIT License](LICENSE).

WinSparkle is an independent upstream project and is distributed under its own license. This action is not affiliated with or endorsed by the WinSparkle maintainers. No WinSparkle binaries are bundled in this repository.
