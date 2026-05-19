## Suggested security fixes — review, then merge if you agree
The security gate **blocked** [this scan](https://github.com/davidayomide/VAmPI/actions/runs/26072184748). This branch carries **AI-generated suggested fixes** for the findings, so you can review a concrete diff and decide. They are a starting point, not a guarantee — **review every line**.

> 🔒 **This PR is NEVER auto-merged.** Merging it is *your* explicit acceptance of these changes; the merge commit re-runs the security gate (and your own CI/tests).

### ✅ Suggested fixes on this branch (2)
- `models/user_model.py:60-67 (A03:2021)`
  - why: The get_user static method constructs a raw SQL query by directly interpolating the username parameter into an f-string without any sanitisation or parameterisation. When vuln is truthy (the default), an attacker can inject arbitrary SQL through the username path parameter. Inte…
  - intentionally minimal; verify behaviour before merge.
- `config.py:9 (A02:2021)`
  - why: The JWT signing secret key is hardcoded as the trivially guessable string 'random'. Any party who reads the source code (public repository) or brute-forces the short key can forge arbitrary JWT tokens, impersonate any user including admins, and bypass all token-based authenticat…
  - ⚠️ **sensitive (manual-review category (A02:2021); protected path; touches SECRET_KEY)** — review this one extra carefully before accepting.

### 🔑 Hardcoded secrets — remove & ROTATE (NOT auto-edited)
These files contain hardcoded credentials. This PR does **not** change them — a wrong automated edit to a credential can break startup and the secret stays in git history anyway. For each:

1. Delete the hardcoded literal from source.
2. Read it at runtime from **1Password** — e.g. an `op://<vault>/<item>/<field>` secret reference resolved by the 1Password CLI / a 1Password Connect or Service-Account token — never from committed source.
3. **Rotate** the exposed credential now (assume it is compromised the moment it was committed).
- `api_views/users.py`
- `config.py`
- `models/user_model.py`
- `openapi_specs/openapi3.yml`

### 📝 Couldn't be applied cleanly — apply manually (6)
- `api_views/books.py:42-54 (A01:2021) — no usable drop-in (see suggestion)`
- `api_views/users.py:113-122 (A01:2021) — no usable drop-in (see suggestion)`
- `api_views/users.py:30-42 (A07:2021) — no usable drop-in (see suggestion)`
- `api_views/users.py:22-24 (A05:2021) — no usable drop-in (see suggestion)`
- `.github/workflows/security-scan.yml:22 (A08:2021) — sketch, not auto-applied`
- `app.py:14 (A05:2021) — syntax error after fix, not auto-applied`

_These had no safe drop-in (no/sketchy code, bad line range, too large, would introduce a new issue, or wouldn't compile). The full suggested fix for each is in `SECURITY-REVIEW.md` on this branch and the `security-scan-report` artifact._

---
> ⚠️ **Review every change. This PR is never auto-merged.** Merging is *your* decision to accept these AI suggestions; the merge commit re-runs the security gate and your own CI/tests.

### ⚠️ Automated self-check advisories
These did **not** block this PR — *you* are the reviewer. Weigh them:

- ⚠️ ruff/bandit flagged a NEW lint/security issue in a suggested fix — review carefully
