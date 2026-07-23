# Application Manifests and Automatic Updates Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add working Windows x64 Scoop manifests for Orca, Open Design, and Dyad, document them in English, and update them automatically from stable GitHub Releases.

**Architecture:** Use Scoop's native `checkver` and `autoupdate` fields because the upstream Windows assets are installers or a NuGet ZIP rather than the ZIP layout required by `make-scoop-manifest`. Use the pinned `ScoopInstaller/GithubActions` Excavator action to run those fields on a four-hour schedule and commit successful changes directly to `main`.

**Tech Stack:** Scoop manifest JSON, PowerShell manifest hooks, GitHub Actions, Git, 7-Zip

---

### Task 1: Add the Orca Manifest

**Files:**

- Create: `bucket/orca.json`
- Verify: `/tmp/dev-tools-scoop-verify/orca.exe` and `/tmp/dev-tools-scoop-verify/orca-outer/app-64.7z`

- [ ] **Step 1: Write the manifest**

Create `bucket/orca.json` with exactly this content:

````json
{
    "version": "1.4.152",
    "description": "Agent development environment for working with a fleet of parallel coding agents.",
    "homepage": "https://github.com/stablyai/orca",
    "license": "MIT",
    "architecture": {
        "64bit": {
            "url": "https://github.com/stablyai/orca/releases/download/v1.4.152/orca-windows-setup.exe#/dl.7z",
            "hash": "sha512:cbe0157d4d5fb5b07e0e0578559103388ae9829d5dcf4564f491bd9322e0772a84cc87c467865c735147dd7bc591e3d050dc434204880303cbbd15332bb18e00"
        }
    },
    "pre_install": "Expand-7zipArchive \"$dir\\`$PLUGINSDIR\\app-64.7z\" \"$dir\"",
    "post_install": "Get-ChildItem \"$dir\" -Filter '$*' | Remove-Item -Recurse -Force",
    "bin": [
        [
            "resources\\bin\\orca.exe",
            "orca"
        ]
    ],
    "shortcuts": [
        [
            "Orca.exe",
            "Orca"
        ]
    ],
    "checkver": {
        "url": "https://api.github.com/repos/stablyai/orca/releases/latest",
        "jsonpath": "$.tag_name",
        "regex": "^v([\\d.]+)$"
    },
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": "https://github.com/stablyai/orca/releases/download/v$version/orca-windows-setup.exe#/dl.7z",
                "hash": {
                    "url": "$baseurl/latest.yml",
                    "regex": "sha512:\\s+$base64"
                }
            }
        }
    }
}
````

- [ ] **Step 2: Validate JSON and manifest identity**

Run:

```bash
jq empty bucket/orca.json
test "$(jq -r .version bucket/orca.json)" = "1.4.152"
test "$(jq -r .architecture.64bit.hash bucket/orca.json)" = "sha512:cbe0157d4d5fb5b07e0e0578559103388ae9829d5dcf4564f491bd9322e0772a84cc87c467865c735147dd7bc591e3d050dc434204880303cbbd15332bb18e00"
```

Expected: all commands exit 0 with no output.

- [ ] **Step 3: Verify the release hash and archive paths**

Run:

```bash
sha512sum /tmp/dev-tools-scoop-verify/orca.exe
7z l -ba /tmp/dev-tools-scoop-verify/orca.exe | rg '\$PLUGINSDIR/app-64\.7z$'
7z l -ba /tmp/dev-tools-scoop-verify/orca-outer/app-64.7z | rg 'Orca\.exe$|resources/app-update\.yml$|resources/bin/orca\.exe$'
```

Expected: the SHA512 is `cbe0157d4d5fb5b07e0e0578559103388ae9829d5dcf4564f491bd9322e0772a84cc87c467865c735147dd7bc591e3d050dc434204880303cbbd15332bb18e00`; the outer path matches once; all three inner paths match.

- [ ] **Step 4: Commit the Orca manifest**

Run:

```bash
git add bucket/orca.json
git commit -m "feat: add Orca Scoop manifest"
```

Expected: one commit containing only `bucket/orca.json`.

### Task 2: Add the Open Design Manifest

**Files:**

- Create: `bucket/open-design.json`
- Verify: `/tmp/dev-tools-scoop-verify/open-design.exe`, `/tmp/dev-tools-scoop-verify/open-outer/payload-base.7z`, and `/tmp/dev-tools-scoop-verify/open-outer/payload-overlay.7z`

- [ ] **Step 1: Write the manifest**

Create `bucket/open-design.json` with exactly this content:

````json
{
    "version": "0.16.0",
    "description": "Local-first desktop design app powered by coding agents.",
    "homepage": "https://github.com/nexu-io/open-design",
    "license": "Apache-2.0",
    "architecture": {
        "64bit": {
            "url": "https://github.com/nexu-io/open-design/releases/download/open-design-v0.16.0/open-design-0.16.0-win-x64-setup.exe#/dl.7z",
            "hash": "99db28bbb4918954030aeb30412252e198acdfccd4880e1684d7acf8c55ec698"
        }
    },
    "pre_install": [
        "Expand-7zipArchive \"$dir\\`$PLUGINSDIR\\payload-base.7z\" \"$dir\"",
        "Expand-7zipArchive \"$dir\\`$PLUGINSDIR\\payload-overlay.7z\" \"$dir\""
    ],
    "post_install": "Get-ChildItem \"$dir\" -Filter '$*' | Remove-Item -Recurse -Force",
    "shortcuts": [
        [
            "Open Design.exe",
            "Open Design"
        ]
    ],
    "checkver": {
        "url": "https://api.github.com/repos/nexu-io/open-design/releases/latest",
        "jsonpath": "$.tag_name",
        "regex": "^open-design-v([\\d.]+)$"
    },
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": "https://github.com/nexu-io/open-design/releases/download/open-design-v$version/open-design-$version-win-x64-setup.exe#/dl.7z",
                "hash": {
                    "url": "$url.sha256"
                }
            }
        }
    }
}
````

- [ ] **Step 2: Validate JSON and manifest identity**

Run:

```bash
jq empty bucket/open-design.json
test "$(jq -r .version bucket/open-design.json)" = "0.16.0"
test "$(jq -r .architecture.64bit.hash bucket/open-design.json)" = "99db28bbb4918954030aeb30412252e198acdfccd4880e1684d7acf8c55ec698"
```

Expected: all commands exit 0 with no output.

- [ ] **Step 3: Verify the release hash and archive paths**

Run:

```bash
sha256sum /tmp/dev-tools-scoop-verify/open-design.exe
7z l -ba /tmp/dev-tools-scoop-verify/open-design.exe | rg '\$PLUGINSDIR/payload-(base|overlay)\.7z$'
7z l -ba /tmp/dev-tools-scoop-verify/open-outer/payload-base.7z | rg 'resources/open-design/bin/7z\.exe$'
7z l -ba /tmp/dev-tools-scoop-verify/open-outer/payload-overlay.7z | rg 'Open Design\.exe$|resources/open-design-config\.json$'
```

Expected: the SHA256 is `99db28bbb4918954030aeb30412252e198acdfccd4880e1684d7acf8c55ec698`; both outer payload paths, the base payload path, and both overlay paths match.

- [ ] **Step 4: Commit the Open Design manifest**

Run:

```bash
git add bucket/open-design.json
git commit -m "feat: add Open Design Scoop manifest"
```

Expected: one commit containing only `bucket/open-design.json`.

### Task 3: Add the Dyad Manifest

**Files:**

- Create: `bucket/dyad.json`
- Verify: `/tmp/dev-tools-scoop-verify/dyad.nupkg`

- [ ] **Step 1: Write the manifest**

Create `bucket/dyad.json` with exactly this content:

````json
{
    "version": "1.8.0",
    "description": "Local, open-source AI app builder for power users.",
    "homepage": "https://github.com/dyad-sh/dyad",
    "license": "MIT",
    "architecture": {
        "64bit": {
            "url": "https://github.com/dyad-sh/dyad/releases/download/v1.8.0/dyad-1.8.0-full.nupkg",
            "hash": "0954a030adbcd48d93b90cb00c44347605fa801b9c0f74bb1e20c71c86d79e11"
        }
    },
    "extract_dir": "lib\\net45",
    "shortcuts": [
        [
            "dyad.exe",
            "Dyad"
        ]
    ],
    "checkver": {
        "url": "https://api.github.com/repos/dyad-sh/dyad/releases/latest",
        "jsonpath": "$.tag_name",
        "regex": "^v([\\d.]+)$"
    },
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": "https://github.com/dyad-sh/dyad/releases/download/v$version/dyad-$version-full.nupkg"
            }
        }
    }
}
````

- [ ] **Step 2: Validate JSON and manifest identity**

Run:

```bash
jq empty bucket/dyad.json
test "$(jq -r .version bucket/dyad.json)" = "1.8.0"
test "$(jq -r .extract_dir bucket/dyad.json)" = "lib\\net45"
test "$(jq -r .architecture.64bit.hash bucket/dyad.json)" = "0954a030adbcd48d93b90cb00c44347605fa801b9c0f74bb1e20c71c86d79e11"
```

Expected: all commands exit 0 with no output.

- [ ] **Step 3: Verify the NuGet hash and extraction root**

Run:

```bash
sha256sum /tmp/dev-tools-scoop-verify/dyad.nupkg
7z l -ba /tmp/dev-tools-scoop-verify/dyad.nupkg | rg 'lib/net45/dyad.exe$'
7z l -ba /tmp/dev-tools-scoop-verify/dyad.nupkg | rg 'lib/net45/resources/app.asar$'
```

Expected: the SHA256 is `0954a030adbcd48d93b90cb00c44347605fa801b9c0f74bb1e20c71c86d79e11`; both paths are present.

- [ ] **Step 4: Commit the Dyad manifest**

Run:

```bash
git add bucket/dyad.json
git commit -m "feat: add Dyad Scoop manifest"
```

Expected: one commit containing only `bucket/dyad.json`.

### Task 4: Add Excavator Automation and Package Documentation

**Files:**

- Create: `.github/workflows/excavator.yml`
- Modify: `README.md`
- Delete: `bucket/.gitkeep`

- [ ] **Step 1: Add the scheduled workflow**

Create `.github/workflows/excavator.yml` with exactly this content:

````yaml
name: Excavator

on:
  workflow_dispatch:
  schedule:
    - cron: '20 */4 * * *'

permissions:
  contents: write

concurrency:
  group: excavator
  cancel-in-progress: false

jobs:
  excavate:
    name: Excavate
    runs-on: windows-latest
    steps:
      - name: Checkout
        uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6.0.2
      - name: Excavate
        uses: ScoopInstaller/GithubActions@e174c3bef2aeec16a40f2f075cafa167733f0a3e # main
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SKIP_UPDATED: '1'
          THROW_ERROR: '1'
````

- [ ] **Step 2: Replace the README package section**

Replace `README.md` with exactly this content:

````markdown
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
````

- [ ] **Step 3: Remove the directory placeholder**

Run:

```bash
git rm bucket/.gitkeep
```

Expected: Git stages only the deletion of `bucket/.gitkeep`.

- [ ] **Step 4: Validate all manifests and workflow content**

Run:

```bash
jq empty bucket/orca.json
jq empty bucket/open-design.json
jq empty bucket/dyad.json
python3 -c 'import pathlib, yaml; p = yaml.safe_load(pathlib.Path(".github/workflows/excavator.yml").read_text()); trigger = p.get("on", p.get(True)); assert trigger is not None; assert p["permissions"]["contents"] == "write"; assert p["jobs"]["excavate"]["runs-on"] == "windows-latest"'
rg -n "ScoopInstaller/GithubActions@e174c3bef2aeec16a40f2f075cafa167733f0a3e" .github/workflows/excavator.yml
rg -n "THROW_ERROR: '1'" .github/workflows/excavator.yml
rg -n "dev-tools/(orca|open-design|dyad)" README.md
git diff --check
```

Expected: all commands exit 0; the YAML parser sees a workflow trigger and `contents: write`; each manifest name appears in the README; `git diff --check` prints nothing.

- [ ] **Step 5: Confirm this task's scope**

Run:

```bash
git status --short --untracked-files=all
git diff --cached --name-only
```

Expected: `README.md` and `.github/workflows/excavator.yml` are the only unstaged or untracked files; `bucket/.gitkeep` is the only staged deletion. The three manifest files are clean because Tasks 1-3 committed them separately.

- [ ] **Step 6: Commit automation and documentation**

Run:

```bash
git add .github/workflows/excavator.yml README.md
git commit -m "feat: automate Scoop manifest updates"
```

Expected: one commit containing the workflow, README, and the already-staged `.gitkeep` deletion.

### Task 5: Run Final Verification

**Files:**

- Verify: `.github/workflows/excavator.yml`, `README.md`, and `bucket/*.json`

- [ ] **Step 1: Validate current release versions against GitHub**

Run:

```bash
test "$(curl -fsSL https://api.github.com/repos/stablyai/orca/releases/latest | jq -r .tag_name)" = "v$(jq -r .version bucket/orca.json)"
test "$(curl -fsSL https://api.github.com/repos/nexu-io/open-design/releases/latest | jq -r .tag_name)" = "open-design-v$(jq -r .version bucket/open-design.json)"
test "$(curl -fsSL https://api.github.com/repos/dyad-sh/dyad/releases/latest | jq -r .tag_name)" = "v$(jq -r .version bucket/dyad.json)"
```

Expected: all commands exit 0 with no output.

- [ ] **Step 2: Verify download URLs and hashes**

Run:

```bash
curl -fsSIL https://github.com/stablyai/orca/releases/download/v1.4.152/orca-windows-setup.exe
curl -fsSIL https://github.com/nexu-io/open-design/releases/download/open-design-v0.16.0/open-design-0.16.0-win-x64-setup.exe
curl -fsSIL https://github.com/dyad-sh/dyad/releases/download/v1.8.0/dyad-1.8.0-full.nupkg
sha512sum /tmp/dev-tools-scoop-verify/orca.exe
sha256sum /tmp/dev-tools-scoop-verify/open-design.exe
sha256sum /tmp/dev-tools-scoop-verify/dyad.nupkg
```

Expected: each URL responds successfully; hashes match the values recorded in the manifests.

- [ ] **Step 3: Run the Windows Scoop smoke check when a Windows runner is available**

From a Windows PowerShell checkout of this branch, run:

```powershell
scoop install .\bucket\orca.json
scoop install .\bucket\open-design.json
scoop install .\bucket\dyad.json
scoop uninstall orca
scoop uninstall open-design
scoop uninstall dyad
```

Expected: each manifest extracts the documented executable, creates its shortcut without running the upstream installer, and uninstalls cleanly. This is the only platform-specific check; the current Linux workspace cannot execute Windows binaries.

- [ ] **Step 4: Check the final worktree**

Run:

```bash
git diff --check HEAD~4..HEAD
git status --short --branch
```

Expected: no whitespace errors and a clean feature branch. Do not push or merge until the user chooses the branch integration workflow.
