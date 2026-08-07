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
scoop search buzz
scoop search openworker
```

Install packages using the bucket-qualified manifest name:

```powershell
scoop install dev-tools/orca
scoop install dev-tools/open-design
scoop install dev-tools/buzz
scoop install dev-tools/openworker
```

Refresh bucket indexes, then update installed packages. **These two commands do different things — don't confuse them:**

- `scoop update` (no argument) runs `git pull` on every bucket, pulling the latest manifests from this repo. Run this **before installing a newly added package** so your local bucket index is current.
- `scoop update *` or `scoop update <app>` only upgrades already-installed apps and **never** fetches new manifests added to the bucket. If a brand-new package was just added upstream, this command alone will not make it installable.

```powershell
scoop update          # refresh bucket indexes (pulls new manifests) — run first
scoop update *        # upgrade all installed apps to their latest versions
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
| `buzz` | [block/buzz](https://github.com/block/buzz) | Self-hostable workspace where humans and AI agents share the same rooms. |
| `openworker` | [andrewyng/openworker](https://github.com/andrewyng/openworker) | Open-source AI coworker that completes everyday tasks from the desktop. |

Manifests track stable GitHub Releases and are refreshed automatically by GitHub Actions.

## Contributing

Package requests and pull requests are welcome. Projects already available from an official Scoop bucket are not duplicated here. New manifests should reference a stable Windows release asset, include a verifiable hash, and document any archive layout needed by Scoop.

## License

This repository is licensed under the [MIT License](LICENSE).
