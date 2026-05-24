# github-ftp-test

Test repository for deploying files via FTP using GitHub Actions.

## How it works

On every push to `main`, the workflow in `.github/workflows/deploy-ftp.yml` deploys `index.html` to the configured FTP server using [SamKirkland/FTP-Deploy-Action](https://github.com/SamKirkland/FTP-Deploy-Action).

Credentials are stored exclusively as GitHub Secrets — never in code.

See [WORKFLOW_DEBUG.md](WORKFLOW_DEBUG.md) for setup details, troubleshooting notes, and a testing checklist.
