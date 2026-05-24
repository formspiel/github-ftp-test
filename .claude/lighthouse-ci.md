# Lighthouse CI — Session Instructions

You are helping set up or maintain automated Lighthouse CI checks via GitHub Actions.
Follow these instructions exactly.

---

## What Lighthouse CI does

Runs Google Lighthouse against every pull request and fails the check if scores
drop below the configured thresholds. No API key required. Results (with a public
link to the full report) are uploaded to Google's temporary storage for 7 days.

Lighthouse checks four categories:
- **Accessibility** — axe-core based, same engine as axe Linter (WCAG 2.1)
- **Best Practices** — security headers, JS errors, deprecated APIs
- **SEO** — meta tags, crawlability
- **Performance** — Core Web Vitals (less useful for simple static pages)

---

## Step 1 — Create the workflow file

Create `.github/workflows/lighthouse.yml`:

```yaml
name: Lighthouse CI

on:
  pull_request:

jobs:
  lighthouse:
    name: Lighthouse Accessibility & Quality
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v12
        with:
          configPath: ./lighthouserc.json
          temporaryPublicStorage: true
```

---

## Step 2 — Create the config file

Create `lighthouserc.json` at the repo root:

```json
{
  "ci": {
    "collect": {
      "staticDistDir": "./"
    },
    "assert": {
      "assertions": {
        "categories:accessibility": ["error", { "minScore": 1 }]
      }
    },
    "upload": {
      "target": "temporary-public-storage"
    }
  }
}
```

**`staticDistDir: "./"` is the key setting** — it tells Lighthouse to serve the repo
root as a static site. Change this to `"./dist"` or `"./public"` for built frontends.

### Threshold levels

| Level | Effect |
|---|---|
| `"error"` | Fails the CI check |
| `"warn"` | Leaves a warning but does not fail |
| `"off"` | Disables the category entirely |

`minScore` is 0–1 (e.g. `0.9` = 90%).

---

## Step 3 — Commit and push

No secrets required. Commit both files and push. The check runs automatically
on every pull request — it does not run on direct pushes to main.

---

## Step 4 — Test it works

The safest way to confirm the check is working is to introduce a deliberate
accessibility violation in a test branch, open a PR, verify the check fails,
then fix it and verify it goes green.

**Example violation to add temporarily:**
```html
<!-- missing alt attribute — WCAG 1.1.1, Lighthouse will catch this -->
<img src="https://placehold.co/400x200">
```

Remove it once confirmed.

---

## Common adjustments

**Built frontend (Vite, Next.js, etc.):**
```json
"staticDistDir": "./dist"
```

**Scan a specific URL instead of local files:**
Replace `staticDistDir` with:
```json
"url": ["https://your-staging-url.com"]
```

**Make performance a hard failure:**
```json
"categories:performance": ["error", { "minScore": 0.8 }]
```

**Skip a category entirely:**
```json
"categories:seo": ["off"]
```

---

## Hard rules — never break these

1. **Do not add `categories:performance` as `"error"` for static pages without
   a CDN** — scores vary too much between runs and will cause flaky failures.

2. **`staticDistDir` must point to a folder containing at least one `.html` file.**
   If it's empty or wrong, the action silently scans nothing and passes — which is
   a false green.

3. **No API key or GitHub Secret is required.** `temporaryPublicStorage` uses
   Google's public storage. If a future session suggests adding an API key, that
   is for a paid Deque/axe Linter product — not needed here.
