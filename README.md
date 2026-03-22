# DiffSense API

> **AI-powered git diff analysis** â€” Turn any git diff into a conventional commit message, security review, or changelog entry in seconds.

[![RapidAPI](https://img.shields.io/badge/RapidAPI-DiffSense-blue?style=flat-square)](https://rapidapi.com/diffsense/api/diffsense-api)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

---

## What is DiffSense?

DiffSense is a REST API that reads your `git diff` output and returns structured, actionable developer intelligence â€” powered by AI.

Stop writing commit messages manually. Stop missing security issues in code review. Stop spending time on changelog entries.

**Paste a diff. Get results.**

---

## Endpoints

| Endpoint | Method | Output |
|---|---|---|
| `/commit-message` | POST | Conventional commit string |
| `/review` | POST | JSON with security + quality issues |
| `/changelog` | POST | Markdown changelog entry |
| `/health` | GET | API status |

---

## Quick Start

### 1. Get your diff

```bash
# Last commit
git diff HEAD~1

# Staged changes
git diff --cached

# Between branches
git diff main..feature/my-branch
```

### 2. Call the API

```bash
DIFF=$(git diff HEAD~1)

curl -X POST https://diffsense-api.p.rapidapi.com/commit-message \
  -H "Content-Type: application/json" \
  -H "x-rapidapi-key: YOUR_RAPIDAPI_KEY" \
  -H "x-rapidapi-host: diffsense-api.p.rapidapi.com" \
  -d "{\"diff\": \"$DIFF\"}"
```

---

## Examples

### `/commit-message`

**Request:**
```json
{
  "diff": "diff --git a/auth.py b/auth.py\n--- a/auth.py\n+++ b/auth.py\n@@ -10,6 +10,8 @@\n     user = db.query(username)\n+    if not user:\n+        return None\n     token = jwt.encode({'user': username}, SECRET_KEY)"
}
```

**Response:**
```json
{
  "commit_message": "fix(auth): return None when user is not found in login"
}
```

---

### `/review`

**Request:**
```json
{
  "diff": "diff --git a/api.py b/api.py\n+    query = f\"SELECT * FROM users WHERE id = {user_id}\""
}
```

**Response:**
```json
{
  "security_issues": [
    {
      "severity": "CRITICAL",
      "description": "SQL injection vulnerability â€” user input is directly interpolated into the query string.",
      "line_hint": "query = f\"SELECT * FROM users WHERE id = {user_id}\""
    }
  ],
  "quality_issues": [
    {
      "severity": "MEDIUM",
      "description": "Use parameterized queries or an ORM instead of string formatting.",
      "line_hint": "query = f\"SELECT * FROM users WHERE id = {user_id}\""
    }
  ],
  "summary": "Critical SQL injection risk detected. Immediate remediation required before merging."
}
```

---

### `/changelog`

**Request:**
```json
{
  "diff": "diff --git a/payments.py b/payments.py\n+def retry_failed_payment(order_id, max_retries=3):\n+    ..."
}
```

**Response:**
```json
{
  "changelog": "### Added\n- Automatic retry logic for failed payments with configurable max attempts (default: 3)."
}
```

---

## Use Cases

- **Pre-commit hooks** â€” Auto-generate commit messages before every commit
- **PR bots** â€” Post security reviews as GitHub PR comments automatically
- **GitHub Actions** â€” Integrate into CI/CD to catch issues before merge
- **IDE extensions** â€” Surface commit suggestions directly in your editor
- **Release automation** â€” Auto-generate CHANGELOG.md on every release

---

## Integrations

### Pre-commit hook

Save as `.git/hooks/prepare-commit-msg` and make it executable:

```bash
#!/bin/bash
DIFF=$(git diff --cached)
if [ -z "$DIFF" ]; then exit 0; fi

MSG=$(curl -s -X POST https://diffsense-api.p.rapidapi.com/commit-message \
  -H "Content-Type: application/json" \
  -H "x-rapidapi-key: YOUR_RAPIDAPI_KEY" \
  -H "x-rapidapi-host: diffsense-api.p.rapidapi.com" \
  -d "{\"diff\": $(echo "$DIFF" | jq -Rs .)}" | jq -r '.commit_message')

echo "$MSG" > "$1"
```

### GitHub Actions

```yaml
name: DiffSense Review
on: [pull_request]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 2

      - name: Get diff
        id: diff
        run: echo "diff=$(git diff HEAD~1 | jq -Rs .)" >> $GITHUB_OUTPUT

      - name: DiffSense security review
        run: |
          curl -s -X POST https://diffsense-api.p.rapidapi.com/review \
            -H "Content-Type: application/json" \
            -H "x-rapidapi-key: ${{ secrets.RAPIDAPI_KEY }}" \
            -H "x-rapidapi-host: diffsense-api.p.rapidapi.com" \
            -d "{\"diff\": ${{ steps.diff.outputs.diff }}}"
```

### Python

```python
import subprocess
import requests

diff = subprocess.check_output(["git", "diff", "HEAD~1"]).decode()

response = requests.post(
    "https://diffsense-api.p.rapidapi.com/review",
    headers={
        "Content-Type": "application/json",
        "x-rapidapi-key": "YOUR_RAPIDAPI_KEY",
        "x-rapidapi-host": "diffsense-api.p.rapidapi.com",
    },
    json={"diff": diff}
)

review = response.json()
for issue in review.get("security_issues", []):
    print(f"[{issue['severity']}] {issue['description']}")
```

### Node.js

```javascript
const { execSync } = require("child_process");

const diff = execSync("git diff HEAD~1").toString();

const response = await fetch("https://diffsense-api.p.rapidapi.com/commit-message", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "x-rapidapi-key": "YOUR_RAPIDAPI_KEY",
    "x-rapidapi-host": "diffsense-api.p.rapidapi.com",
  },
  body: JSON.stringify({ diff }),
});

const { commit_message } = await response.json();
console.log(commit_message);
// â†’ "feat(auth): add null check for missing user on login"
```

---

## Limits

| Parameter | Value |
|---|---|
| Max diff size | 8,000 characters (auto-truncated) |
| Max request body | 100 KB |
| Typical response time | < 3 seconds |
| Supported languages | Any (language-agnostic) |

---

## Pricing

Available on [RapidAPI](https://rapidapi.com/diffsense/api/diffsense-api):

| Plan | Price | Requests |
|---|---|---|
| Free | $0/mo | 50 req/mo |
| Pro | $9/mo | 1,000 req/mo |
| Ultra | $29/mo | 10,000 req/mo |

---

## License

MIT
