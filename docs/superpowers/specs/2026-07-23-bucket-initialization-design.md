# Scoop Bucket Initialization Design

## Goal

Initialize a public Scoop bucket with the smallest useful repository structure and English documentation for a global audience.

## Repository Structure

```text
.
|-- bucket/
|   `-- .gitkeep
|-- LICENSE
`-- README.md
```

`bucket/` will contain Scoop JSON manifests. `.gitkeep` keeps the directory in Git until the first manifest is added.

## Documentation

`README.md` will:

- identify the repository as a Scoop bucket for developer tools;
- show how to add it as `dev-tools`;
- show the standard Scoop search and install commands;
- state that manifests will be generated from GitHub Releases with `hymkor/make-scoop-manifest` where applicable;
- briefly explain how to request or contribute a package.

The bucket installation command will use the canonical repository URL:

```powershell
scoop bucket add dev-tools https://github.com/RainbowMine/Dev-Tools-Scoop-Bucket
```

## Manifest Flow

No manifest is included during initialization. When the first supported application is selected, its manifest will be generated from its GitHub Releases assets with `make-scoop-manifest`, reviewed, and added under `bucket/`.

## Error Handling

There is no runtime code in this phase. The documentation will avoid claiming that a package is available before a manifest exists. Future generated manifests must only reference release assets compatible with Scoop on Windows.

## Verification

- Confirm that all documentation is in English.
- Confirm that the repository URL and Scoop commands are valid and internally consistent.
- Confirm that `bucket/` is tracked by Git.
- Check Markdown structure manually.

## Out of Scope

- Package manifests
- GitHub Actions
- Update scripts or configuration
- JSON validation tooling
- New dependencies

Automation will be considered after at least one package exists and recurring maintenance justifies it.
