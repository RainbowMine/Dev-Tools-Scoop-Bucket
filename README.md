# Dev Tools Scoop Bucket

A community-maintained [Scoop](https://scoop.sh/) bucket for developer tools on Windows.

## Usage

Install [Scoop](https://scoop.sh/) on Windows, then add this bucket:

```powershell
scoop bucket add dev-tools https://github.com/RainbowMine/Dev-Tools-Scoop-Bucket
```

Search for a package:

```powershell
scoop search orca
scoop search open-design
```

Install packages using the bucket-qualified manifest name:

```powershell
scoop install dev-tools/orca
scoop install dev-tools/open-design
```

Refresh bucket manifests and update installed packages:

```powershell
scoop update
scoop update orca
scoop update open-design
```

Remove the bucket when it is no longer needed:

```powershell
scoop bucket rm dev-tools
```

## Packages

All packages in this bucket currently target Windows x64.

| Package | Project | Description |
| --- | --- | --- |
| `orca` | [stablyai/orca](https://github.com/stablyai/orca) | Agent development environment for parallel coding agents. |
| `open-design` | [nexu-io/open-design](https://github.com/nexu-io/open-design) | Local-first desktop design app powered by coding agents. |

Manifests track stable GitHub Releases and are refreshed automatically by GitHub Actions.

## Contributing

Package requests and pull requests are welcome. Projects already available from an official Scoop bucket are not duplicated here. New manifests should reference a stable Windows release asset, include a verifiable hash, and document any archive layout needed by Scoop.

## License

This repository is licensed under the [MIT License](LICENSE).
