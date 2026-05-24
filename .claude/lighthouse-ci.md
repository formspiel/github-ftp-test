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

## Configuration options — choose what fits the project

When setting up for a new repo, ask the user which configuration fits best
and adjust `lighthouserc.json` accordingly. Present these options:

---

### Option 1 — Accessibility only, 100% (current default)

Strictest accessibility gate. Any single violation blocks the merge.
Best for projects where accessibility is the primary concern.

```json
{
  "ci": {
    "collect": { "staticDistDir": "./" },
    "assert": {
      "assertions": {
        "categories:accessibility": ["error", { "minScore": 1 }]
      }
    },
    "upload": { "target": "temporary-public-storage" }
  }
}
```

---

### Option 2 — Accessibility only, 90%

Slightly more lenient — allows minor issues through. Good for projects
that are still being improved toward full compliance.

```json
{
  "ci": {
    "collect": { "staticDistDir": "./" },
    "assert": {
      "assertions": {
        "categories:accessibility": ["error", { "minScore": 0.9 }]
      }
    },
    "upload": { "target": "temporary-public-storage" }
  }
}
```

---

### Option 3 — Accessibility + Best Practices + SEO

Broader quality gate. Accessibility blocks the merge; the others warn
but don't fail. Good for projects where overall quality matters.

```json
{
  "ci": {
    "collect": { "staticDistDir": "./" },
    "assert": {
      "assertions": {
        "categories:accessibility": ["error", { "minScore": 1 }],
        "categories:best-practices": ["warn", { "minScore": 0.8 }],
        "categories:seo": ["warn", { "minScore": 0.8 }]
      }
    },
    "upload": { "target": "temporary-public-storage" }
  }
}
```

---

### Option 4 — Everything as hard failures

Maximum strictness. All four categories must meet their threshold or
the merge is blocked. Only use this for projects with stable, predictable
performance scores (i.e. served via CDN, not raw GitHub Pages).

```json
{
  "ci": {
    "collect": { "staticDistDir": "./" },
    "assert": {
      "assertions": {
        "categories:accessibility": ["error", { "minScore": 1 }],
        "categories:best-practices": ["error", { "minScore": 0.9 }],
        "categories:seo": ["error", { "minScore": 0.9 }],
        "categories:performance": ["error", { "minScore": 0.8 }]
      }
    },
    "upload": { "target": "temporary-public-storage" }
  }
}
```

⚠️ See hard rule #1 before enabling performance as an error.

---

### Adjusting the source directory

| Project type | Setting |
|---|---|
| Static HTML at repo root | `"staticDistDir": "./"` |
| Built frontend (Vite, Next.js) | `"staticDistDir": "./dist"` |
| Custom public folder | `"staticDistDir": "./public"` |
| Remote staging URL | replace `staticDistDir` with `"url": ["https://staging.example.com"]` |

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
