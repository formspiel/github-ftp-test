# Claude Code Session Instructions

## FTP Deployment — Lessons Learned

### 1. Use the correct action version (with `v` prefix)

When referencing `SamKirkland/FTP-Deploy-Action`, the tag format requires a `v` prefix:

```yaml
# WRONG — tag does not exist without the v prefix
uses: SamKirkland/FTP-Deploy-Action@4.3.5

# CORRECT
uses: SamKirkland/FTP-Deploy-Action@v4.4.0
```

Always verify the latest tag at https://github.com/SamKirkland/FTP-Deploy-Action/releases before writing the workflow.

### 2. Use `protocol: ftps`, not `protocol: ftp`

Plain `ftp` causes an `ECONNRESET` on the data socket when the GitHub Actions runner connects via IPv6 (EPSV mode). The KAS server at `w01cec68.kasserver.com` supports `AUTH TLS`, so always use:

```yaml
protocol: ftps
```

This uses explicit TLS over port 21 and avoids the IPv6 data socket reset entirely.

### 3. Credentials go in GitHub Secrets only

Never hardcode FTP credentials. Reference them exclusively via:

```yaml
server:   ${{ secrets.FTP_SERVER }}
username: ${{ secrets.FTP_USERNAME }}
password: ${{ secrets.FTP_PASSWORD }}
port:     ${{ secrets.FTP_PORT }}
```

Required secrets: `FTP_SERVER`, `FTP_USERNAME`, `FTP_PASSWORD`, `FTP_PORT`

---

## Workflow Trigger Reference

### Current trigger

```yaml
on:
  push:
    branches:
      - ready
```

The workflow fires on every push to the `ready` branch.

### Other useful triggers

```yaml
# Manual trigger from the GitHub Actions UI (no code push needed)
on:
  workflow_dispatch:

# Multiple branches
on:
  push:
    branches:
      - main
      - ready
      - release/**

# Pull request (fires on open, sync, reopen — NOT on merge to target)
on:
  pull_request:
    branches:
      - main

# Scheduled (cron) — e.g. every day at 08:00 UTC
on:
  schedule:
    - cron: '0 8 * * *'

# On published GitHub release
on:
  release:
    types: [published]

# Combine triggers
on:
  push:
    branches: [ready]
  workflow_dispatch:
```

`workflow_dispatch` is especially useful for testing — it lets you trigger the workflow manually from the Actions tab without pushing a commit.
