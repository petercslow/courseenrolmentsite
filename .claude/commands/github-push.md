---
description: Pre-flight safety checks, refresh README, wire up GitHub Pages via Actions, commit, push, and configure repo metadata
argument-hint: "[commit message]"
allowed-tools: Bash(git:*), Bash(gh:*), Read, Write, Edit, Glob, Grep, SlashCommand
---

Perform a full "ready to publish" push of this repository to GitHub, ending with a live GitHub Pages URL. Work through the steps below in order and stop to tell the user if any step fails or looks unsafe — do not silently skip a failure and continue.

## 1. Pre-flight safety checks

- Run `git status --porcelain` and `git diff` (plus `git diff --staged`) to see exactly what would be committed. Read the actual file contents for anything unfamiliar or binary before staging it.
- **`.gitignore`**: if none exists at the repo root, create one with sensible defaults for this kind of project (OS cruft: `.DS_Store`, `Thumbs.db`; editor dirs: `.vscode/`, `.idea/`; env/secret files: `.env`, `.env.*`, `*.pem`, `*.key`, `credentials*.json`; and `node_modules/`, `dist/`, `build/` in case tooling is added later). If one already exists, check it covers env/secret/OS patterns and append any that are missing — don't rewrite unrelated entries.
- **Secret scan**: grep the files that are about to be added/committed (from `git status --porcelain`, both staged and untracked) for high-signal secret patterns before anything is staged:
  - Private key headers: `-----BEGIN (RSA |EC |OPENSSH )?PRIVATE KEY-----`
  - Cloud/provider tokens: `AKIA[0-9A-Z]{16}` (AWS), `ghp_`, `gho_`, `ghu_`, `ghs_`, `ghr_` (GitHub), `xox[baprs]-` (Slack), `sk-[A-Za-z0-9]{20,}` (generic API-key shape)
  - Generic assignments with a non-placeholder-looking value: `(api[_-]?key|secret|token|password)\s*[:=]\s*['"][^'"]{8,}['"]`
  - If any match is found, **stop immediately**, show the user the file and matched line, and do not stage, commit, or push until they've confirmed it's safe or removed it.
- Confirm GitHub CLI auth: `gh auth status`. If not authenticated, stop and tell the user to run `gh auth login` first — do not attempt to work around this.

## 2. Refresh README.md

Invoke the project's `/readme` slash command (via the SlashCommand tool) to regenerate `README.md` from the current state of the code. If this project has no `/readme` command available, tell the user plainly that it's missing and skip this step — do not fabricate README content yourself as a substitute for the intended command.

## 3. Add the GitHub Pages Actions workflow

Create (or overwrite) `.github/workflows/deploy.yml` with a workflow that deploys the static site via GitHub Actions on every push to `main`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure Pages
        uses: actions/configure-pages@v5

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: .

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

If the project already has a build step by the time this command runs (e.g. a bundler was introduced), add the appropriate install/build steps before `Upload artifact` and point `path:` at the build output directory instead of `.` — don't assume this blindly, check for a `package.json`/build config first.

## 4. Commit and push

- Stage the intended changes explicitly (README, `.gitignore`, the new workflow file, and anything else genuinely part of this change) — avoid a blind `git add -A` if the earlier `git status` showed unrelated untracked files.
- Show the user a short summary of what's staged (`git status`) before committing.
- Commit with a clear, specific message describing what changed (e.g. README refresh + GitHub Pages Actions workflow), using `$ARGUMENTS` as the message if the user supplied one, otherwise writing your own descriptive message. Follow this repo's commit attribution convention if one is in effect for the session.
- Push to `origin` on the current branch's upstream (`git push`), or `origin main` if no upstream is set yet.

## 5. Enable GitHub Pages (build via Actions)

Resolve `owner/repo` with `gh repo view --json nameWithOwner -q .nameWithOwner`, then enable Pages with the workflow build type:

```
gh api -X POST repos/{owner}/{repo}/pages -f build_type=workflow
```

If Pages is already enabled for this repo, that call returns a 422 — in that case update the existing configuration instead:

```
gh api -X PUT repos/{owner}/{repo}/pages -f build_type=workflow
```

## 6. Set repo metadata

Use `gh repo edit` to set the description, homepage, and topics so the repo page is presentable:

```
gh repo edit --description "<concise one-line description of the project>" \
  --homepage "https://{owner}.github.io/{repo}/" \
  --add-topic <topic1> --add-topic <topic2> ...
```

Base the description and topics on the project's actual README/content (don't invent unrelated ones) — e.g. for a static training-enrolment prototype, topics like `static-site`, `vanilla-javascript`, `github-pages`, and a domain topic (e.g. `education` or `training`) are reasonable; adjust to fit whatever project this command is actually run in.

## 7. Report the live URL

Fetch and print the Pages URL:

```
gh api repos/{owner}/{repo}/pages --jq .html_url
```

Tell the user this URL, note that the first deploy can take a minute or two after the workflow run finishes, and point them at the running workflow (`gh run list --workflow=deploy.yml -L 1`) so they can watch its progress if they want to.
