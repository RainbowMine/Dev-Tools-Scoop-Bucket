# Dev Tools Scoop Bucket

A community-maintained [Scoop](https://scoop.sh/) bucket for developer tools on Windows.

## Usage

Add the bucket:

```powershell
scoop bucket add dev-tools https://github.com/RainbowMine/Dev-Tools-Scoop-Bucket
```

Update Scoop and search for a package:

```powershell
scoop update
scoop search <package>
```

Install a package:

```powershell
scoop install dev-tools/orca
```

## Packages

All packages in this bucket currently target Windows x64.

| Package | Project | Description |
| --- | --- | --- |
| `orca` | [stablyai/orca](https://github.com/stablyai/orca) | Agent development environment for parallel coding agents. |
| `open-design` | [nexu-io/open-design](https://github.com/nexu-io/open-design) | Local-first desktop design app powered by coding agents. |
| `dyad` | [dyad-sh/dyad](https://github.com/dyad-sh/dyad) | Local, open-source AI app builder for power users. |

Install any package with its manifest name, for example:

```powershell
scoop install dev-tools/orca
scoop install dev-tools/open-design
scoop install dev-tools/dyad
```

Manifests track stable GitHub Releases and are refreshed automatically by GitHub Actions.

## Contributing

Package requests and pull requests are welcome. New manifests should reference a stable Windows release asset, include a verifiable hash, and document any archive layout needed by Scoop.

## License

This repository is licensed under the [MIT License](LICENSE).
