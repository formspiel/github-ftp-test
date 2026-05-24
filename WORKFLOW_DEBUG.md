# FTP Deployment Workflow Debug Notes

## Workflow File

Location: `.github/workflows/deploy-ftp.yml`

### Configuration Summary

| Setting | Value |
|---|---|
| Trigger | Push to `main` branch |
| Action | `SamKirkland/FTP-Deploy-Action@v4.4.0` |
| FTP server | `${{ secrets.FTP_SERVER }}` (w01*****.kasserver.com) |
| FTP username | `${{ secrets.FTP_USERNAME }}` (f018508e) |
| FTP port | `${{ secrets.FTP_PORT }}` (21) |
| FTP mode | Passive (action default) |
| Target directory | `/` (server root) |
| Local source | `./` (repo root) |
| Log level | `verbose` |

### Excluded from deployment

- `.git` directory and all git metadata
- `node_modules`
- `.github` directory (workflow files themselves)
- `WORKFLOW_DEBUG.md` (this file)

---

## Required GitHub Secrets

Configure these under **Settings → Secrets and variables → Actions**:

| Secret name | Description |
|---|---|
| `FTP_SERVER` | FTP hostname |
| `FTP_USERNAME` | FTP account username |
| `FTP_PASSWORD` | FTP account password (provided securely) |
| `FTP_PORT` | FTP port (21) |

**Security note:** Never commit credentials to the repository. All sensitive values must remain in GitHub Secrets only.

---

## Steps Taken

1. Created `index.html` — simple HTML test page as the deployment artifact.
2. Created `.github/workflows/deploy-ftp.yml` — GitHub Actions workflow using `SamKirkland/FTP-Deploy-Action@4.3.5`.
3. All credentials referenced via `${{ secrets.* }}` — no hardcoded values.

---

## Testing Checklist

- [ ] GitHub Secrets are configured (FTP_SERVER, FTP_USERNAME, FTP_PASSWORD, FTP_PORT)
- [ ] Push to `main` branch triggers the workflow
- [ ] Check the **Actions** tab in GitHub for live logs
- [ ] Confirm `index.html` appears at the expected URL on the FTP server

---

## Common Errors & What to Check

| Error | Likely cause | Fix |
|---|---|---|
| `ECONNRESET` on data socket | Runner connected via IPv6-mapped socket; kasserver drops IPv6 data connections | Add **Disable IPv6** step (see below) |
| `Login incorrect` | FTP_USERNAME or FTP_PASSWORD secret is wrong | Double-check secrets in repo settings |
| `Connection refused` / timeout | FTP_SERVER hostname or FTP_PORT is wrong | Check firewall and hosting panel |
| `550 Permission denied` | FTP user lacks write access to the target directory | Adjust `server-dir` path |
| `No files uploaded` | Wrong `local-dir` or overly broad `exclude` patterns | Review workflow config |

### ECONNRESET — root cause and fix

**Root cause:** Node.js on some GitHub Actions runners connects via an IPv4-mapped
IPv6 socket (`::ffff:x.x.x.x`). The `basic-ftp` library reads `socket.remoteFamily`,
gets `'IPv6'`, and uses EPSV for the data connection. Kasserver drops IPv6 data
connections, so the upload fails with ECONNRESET.

**Why it's non-deterministic:** runners differ in whether IPv6 is active. Runs on
IPv4-only runners succeed silently; runs on IPv6-enabled runners fail. This is why
earlier test runs may have passed.

**Fix:** disable IPv6 on the runner before the deploy step:

```yaml
- name: Disable IPv6
  run: |
    sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
    sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1
```

**Does NOT fix it:** editing `/etc/gai.conf` — that only changes DNS address-selection
preference, not the socket address family that `basic-ftp` reads.

---

## Recommendations for Next Steps

1. **Verify secrets** — double-check FTP_SERVER, FTP_USERNAME, and FTP_PORT match the hosting panel exactly.
2. **Confirm target directory** — the workflow deploys to `/`. Adjust `server-dir` if the web root is a subdirectory (e.g. `/www/` or `/htdocs/`).
3. **Review verbose logs** — with `log-level: verbose` the Actions log will show every file checked, skipped, or uploaded.
4. **Test with dry-run first** — set `dry-run: true` in the workflow to simulate a deployment without writing any files, then switch back to `false` once the connection is confirmed working.
