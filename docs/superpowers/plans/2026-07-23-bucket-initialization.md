# Scoop Bucket Initialization Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Initialize a public Scoop bucket with a tracked manifest directory and concise English documentation for global users.

**Architecture:** Keep the repository static: Scoop reads future JSON manifests from `bucket/`, while `README.md` documents the public interface. Do not add runtime code, automation, configuration, or dependencies until the first package creates a concrete need.

**Tech Stack:** Scoop, Markdown, Git

---

### Task 1: Create the Public Bucket Skeleton

**Files:**

- Create: `README.md`
- Create: `bucket/.gitkeep`

- [ ] **Step 1: Create the English README**

Create `README.md` with exactly this content:

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

Install a package after it becomes available:

```powershell
scoop install dev-tools/<package>
```

## Packages

No packages are available yet.

## Contributing

Package requests and pull requests are welcome. When a project publishes compatible Windows ZIP assets through GitHub Releases, its manifest can be generated with [hymkor/make-scoop-manifest](https://github.com/hymkor/make-scoop-manifest) and reviewed before inclusion.

## License

This repository is licensed under the [MIT License](LICENSE).
````

- [ ] **Step 2: Track the empty manifest directory**

Create `bucket/.gitkeep` as an empty file. Do not add a placeholder JSON manifest because Scoop users must not see a package that cannot be installed.

- [ ] **Step 3: Verify the repository structure**

Run:

```bash
test -f README.md
test -f bucket/.gitkeep
```

Expected: both commands exit with status 0 and produce no output.

- [ ] **Step 4: Verify required documentation content**

Run:

```bash
rg -n "scoop bucket add dev-tools https://github.com/RainbowMine/Dev-Tools-Scoop-Bucket" README.md
rg -n "scoop search <package>" README.md
rg -n "scoop install dev-tools/<package>" README.md
rg -n "hymkor/make-scoop-manifest" README.md
git diff --check
```

Expected: each `rg` command prints one matching line, and `git diff --check` exits with status 0 and no output.

- [ ] **Step 5: Confirm the scope remains minimal**

Run:

```bash
git status --short
```

Expected: only `README.md` and `bucket/.gitkeep` are new implementation files. Existing design and plan documents may already be tracked on the branch. No workflow, script, configuration, manifest, or dependency file is added.

- [ ] **Step 6: Commit the bucket skeleton**

Run:

```bash
git add README.md bucket/.gitkeep
git commit -m "feat: initialize Scoop bucket"
```

Expected: Git creates one commit containing only `README.md` and `bucket/.gitkeep`.
