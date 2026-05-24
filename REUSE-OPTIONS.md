# How to Reuse the GitHub FTP Deploy Instructions Across Projects

## The Problem

`CLAUDE.md` at a project root is that project's memory file for Claude Code sessions.
Keeping FTP deploy instructions there risks overwriting a project's existing `CLAUDE.md`.

---

## Option A — `.claude/github-ftp-deploy.md` per project (Recommended)

Store the instructions as a named file inside a `.claude/` subdirectory.
Any project can pull it in by adding one line to its existing `CLAUDE.md`:

```markdown
@.claude/github-ftp-deploy.md
```

Claude Code merges imported files automatically — the existing `CLAUDE.md` is untouched.

**To-do for each new project:**
- [ ] Copy `github-ftp-deploy.md` into the project's `.claude/` folder
- [ ] Add `@.claude/github-ftp-deploy.md` to the project's `CLAUDE.md` (create one if it doesn't exist)
- [ ] Add the four GitHub Secrets (FTP_SERVER, FTP_USERNAME, FTP_PASSWORD, FTP_PORT)
- [ ] Copy `.github/workflows/deploy-ftp.yml` from this repo and adjust the trigger branch

---

## Option B — Global `~/.claude/CLAUDE.md`

Add the instructions once to your personal Claude Code config file.
Claude Code merges this automatically into every session on your machine — no per-project setup.

**To-do (one time only):**
- [ ] Open (or create) `~/.claude/CLAUDE.md`
- [ ] Paste the contents of `CLAUDE.md` from this repo into it
- [ ] Per project: still add the four GitHub Secrets and copy the workflow file

**Trade-off:** The instructions are always loaded, even in projects that have nothing to do with FTP.

---

## Option C — Template repo / gist

Keep `formspiel/github-ftp-test` as the canonical source.
For each new project, manually copy the relevant files.

**To-do for each new project:**
- [ ] Copy `.github/workflows/deploy-ftp.yml` and adjust the trigger branch
- [ ] Copy `CLAUDE.md` (or paste its contents into the existing one)
- [ ] Add the four GitHub Secrets

**Trade-off:** No automation, but clean separation. Good if projects differ enough that you'd customise the workflow anyway.

---

## Quick Comparison

| | Option A | Option B | Option C |
|---|---|---|---|
| Per-project setup | Copy 1 file + 1 line | None after first time | Copy files manually |
| Risk of polluting unrelated projects | No | Yes (global) | No |
| Version-controlled per project | Yes | No | Yes |
| Best for | Most projects | Frequent FTP deploys | One-off / custom setups |

---

## What Every Option Requires (GitHub Secrets)

Regardless of which option you choose, each repository needs these four secrets
configured under **Settings → Secrets and variables → Actions**:

| Secret | Value |
|---|---|
| `FTP_SERVER` | `w01cec68.kasserver.com` |
| `FTP_USERNAME` | `f018508e` |
| `FTP_PASSWORD` | *(provide securely)* |
| `FTP_PORT` | `21` |
