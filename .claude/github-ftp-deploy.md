# GitHub FTP Deployment — Session Instructions

You are helping set up or maintain automated FTP deployment via GitHub Actions.
Follow these instructions exactly. They encode hard-won lessons from real failures.

---

## Step 1 — Ask about the trigger branch

Before writing any workflow file, ask the user:

> "The workflow will trigger on push to `main` by default.
> Does that fit this project, or would a different branch work better?
> Common alternatives:
> - `main` — deploy every merge to main (standard for simple sites)
> - `ready` — a dedicated deploy branch; push here only when you want to ship
> - `release/**` — deploy on any release branch
> - `workflow_dispatch` only — manual trigger from the Actions tab, no automatic deploys
> Which fits this project best?"

Wait for the user's answer before proceeding. Use their chosen branch in the workflow below.

---

## Step 2 — Create the workflow file

Create `.github/workflows/deploy-ftp.yml` with the following content,
substituting `BRANCH` with the branch the user chose in Step 1:

```yaml
name: Deploy via FTP

on:
  push:
    branches:
      - BRANCH

jobs:
  deploy:
    name: FTP Deploy
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Deploy to FTP server
        uses: SamKirkland/FTP-Deploy-Action@v4.4.0
        with:
          server: ${{ secrets.FTP_SERVER }}
          username: ${{ secrets.FTP_USERNAME }}
          password: ${{ secrets.FTP_PASSWORD }}
          port: ${{ secrets.FTP_PORT }}
          protocol: ftps
          server-dir: /
          local-dir: ./
          exclude: |
            **/.git*
            **/.git*/**
            **/node_modules/**
            .github/**
          log-level: verbose
          dry-run: false
```

---

## Step 3 — Confirm GitHub Secrets are configured

Tell the user:

> "Before the workflow can run, four secrets must be added to this repository under
> **Settings → Secrets and variables → Actions**:"

| Secret name | Description |
|---|---|
| `FTP_SERVER` | FTP hostname (e.g. `w01*****.kasserver.com`) |
| `FTP_USERNAME` | FTP account username |
| `FTP_PASSWORD` | FTP account password |
| `FTP_PORT` | FTP port (usually `21`) |

Ask: "Are the secrets already configured, or do you need to add them now?"

If they need to add them, wait for confirmation before proceeding.

---

## Step 4 — Commit and push

Commit the workflow file and push to the repository.
If the trigger branch doesn't exist yet, create it.

---

## Step 5 — Verify the run

After pushing, monitor the Actions tab. Common errors and their fixes:

| Error | Fix |
|---|---|
| `unable to find version` | The action tag is wrong — always use `v` prefix: `@v4.4.0` |
| `ECONNRESET` on data socket | Change `protocol: ftp` to `protocol: ftps` — never use plain `ftp` |
| `Login incorrect` | A secret value is wrong — double-check FTP_USERNAME and FTP_PASSWORD |
| `550 Permission denied` | FTP user lacks write access to `server-dir` — adjust the path |
| `No files uploaded` | Check `local-dir` and `exclude` patterns |

---

## Hard rules — never break these

1. **Always use `protocol: ftps`**, never `protocol: ftp`.
   Plain FTP causes `ECONNRESET` on the data socket when the runner connects via IPv6.

2. **Always use the `v` prefix on the action tag**: `@v4.4.0`, not `@4.3.5`.
   Tags without `v` do not exist and will cause "unable to find version" errors.
   Verify the latest release at https://github.com/SamKirkland/FTP-Deploy-Action/releases.

3. **Never hardcode credentials**. Always use `${{ secrets.* }}`.
