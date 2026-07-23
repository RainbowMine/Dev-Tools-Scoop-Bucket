# Application Manifests and Automatic Updates Design

## Goal

Add Scoop manifests for Orca, Open Design, and Dyad, then keep them aligned with each project's latest stable GitHub Release through a scheduled GitHub Actions workflow.

## Constraints

- Support Windows x64 only because none of the three projects currently publishes a Windows ARM64 or x86 release asset.
- Track stable GitHub Releases only. Drafts and prereleases are excluded.
- Commit successful manifest updates directly to `main`.
- Keep project documentation in English.
- Add no custom updater script or runtime dependency.

## Generator Decision

`hymkor/make-scoop-manifest` cannot generate these manifests directly:

- Orca publishes `orca-windows-setup.exe`.
- Open Design publishes `open-design-<version>-win-x64-setup.exe`.
- Dyad publishes a Windows setup executable and `dyad-<version>-full.nupkg`.

The generator requires Windows ZIP release assets with recognizable architecture names. These upstream assets do not meet that contract. The bucket will therefore use Scoop's native `checkver` and `autoupdate` fields, which are the smaller and more reliable solution for installer and NuGet assets.

## Repository Changes

```text
.
|-- .github/
|   `-- workflows/
|       `-- excavator.yml
|-- bucket/
|   |-- dyad.json
|   |-- open-design.json
|   `-- orca.json
`-- README.md
```

`bucket/.gitkeep` will be removed after real manifests exist.

## Orca Manifest

The initial manifest will use stable release `v1.4.152` from `stablyai/orca`.

- Download `orca-windows-setup.exe` as a 7-Zip archive through Scoop's URL fragment syntax.
- Extract the embedded `$PLUGINSDIR/app-64.7z` payload into the Scoop application directory.
- Remove the outer NSIS extraction files after installation.
- Add a Start Menu shortcut for `Orca.exe`.
- Expose `resources/bin/orca.exe` as the `orca` command.
- Declare the MIT license and upstream homepage.
- Use the GitHub latest stable release for version checks.
- Read the upstream SHA512 checksum from `latest.yml` during automatic updates.

## Open Design Manifest

The initial manifest will use stable release `open-design-v0.16.0` from `nexu-io/open-design`.

- Download `open-design-0.16.0-win-x64-setup.exe` as a 7-Zip archive.
- Extract `$PLUGINSDIR/payload-base.7z` and then `$PLUGINSDIR/payload-overlay.7z` into the Scoop application directory.
- Remove the outer NSIS extraction files after installation.
- Add a Start Menu shortcut for `Open Design.exe`.
- Declare the Apache-2.0 license and upstream homepage.
- Extract the version from release tags matching `open-design-v<version>`.
- Read the upstream SHA256 checksum from the matching `.sha256` release asset.

## Dyad Manifest

The initial manifest will use stable release `v1.8.0` from `dyad-sh/dyad`.

- Download `dyad-1.8.0-full.nupkg` directly.
- Extract the `lib/net45` directory into the Scoop application directory.
- Add a Start Menu shortcut for `dyad.exe`.
- Declare the MIT license from the upstream package metadata and link to the upstream homepage.
- Use the GitHub latest stable release for version checks.
- Let Scoop calculate SHA256 for new NuGet assets during automatic updates instead of relying on the upstream Squirrel SHA1 file.

## Application Updaters

The manifests will not patch application binaries or user settings to disable their built-in updaters. Scoop remains responsible for bucket version updates. If Windows installation testing proves that an application's updater modifies or breaks its Scoop-managed directory, that application will receive the smallest targeted mitigation supported by its upstream package format.

## GitHub Actions Workflow

`.github/workflows/excavator.yml` will:

- run every four hours at minute 20;
- support manual `workflow_dispatch` runs;
- grant only `contents: write` repository permission;
- prevent overlapping update jobs with workflow concurrency;
- use immutable commit SHAs for `actions/checkout` and `ScoopInstaller/GithubActions`;
- pass the repository `GITHUB_TOKEN` to Excavator;
- set `SKIP_UPDATED` to reduce unchanged-version output;
- set `THROW_ERROR` so a failed manifest update fails the job.

Excavator will run Scoop's native version checks and autoupdate logic, update versioned URLs and hashes, and commit changed manifests directly to the default branch. Repository Actions settings must allow `GITHUB_TOKEN` write access, and the default branch must permit Action pushes.

## Update Flow

1. The scheduled or manually triggered workflow checks each manifest.
2. `checkver` resolves the latest stable upstream release.
3. If the version changed, `autoupdate` rewrites the URL and obtains or calculates the new hash.
4. Excavator validates the generated manifest.
5. When all processing succeeds, Excavator commits the changed manifest files to `main`.
6. If an asset, checksum, or manifest rule fails, the job fails without intentionally committing a partial update.

## Documentation

`README.md` will replace the empty package notice with an English package table containing the manifest name, upstream project, description, and installation command for:

- `dev-tools/orca`
- `dev-tools/open-design`
- `dev-tools/dyad`

## Verification

- Parse every manifest as JSON.
- Confirm each initial version against GitHub's latest stable Release API.
- Verify initial hashes against upstream checksums or downloaded assets.
- Download and inspect each Windows asset with 7-Zip.
- Confirm the documented inner archive paths and executable names exist.
- Validate `checkver` and `autoupdate` behavior with Scoop on a Windows runner when the workflow is available.
- Parse the workflow YAML and inspect its permissions, triggers, pinned actions, and environment variables.
- Run `git diff --check` and confirm only the requested files changed.

## Out of Scope

- Windows ARM64 or x86 packages
- Prerelease channels
- Pull-request-based automatic updates
- A custom PowerShell updater
- Building or repackaging upstream applications
- Changing application user settings without a demonstrated installation conflict
