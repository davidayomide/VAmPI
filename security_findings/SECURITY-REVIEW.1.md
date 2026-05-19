## Suggested security fixes — review, then merge if you agree
The security gate **blocked** [this scan](https://github.com/davidayomide/VAmPI/actions/runs/26073055841). This branch carries **AI-generated suggested fixes** for the findings, so you can review a concrete diff and decide. They are a starting point, not a guarantee — **review every line**.

> 🔒 **This PR is NEVER auto-merged.** Merging it is *your* explicit acceptance of these changes; the merge commit re-runs the security gate (and your own CI/tests).

### ✅ Suggested fixes on this branch (1)
- `config.py:10 (A02:2021)`
  - why: The JWT signing secret is hard-coded as the trivially guessable string 'random'. Any attacker who knows or guesses this value can forge arbitrary JWTs, impersonate any user including admins, and bypass all token-based authentication. Intentionally unchanged: the rest of the Flas…
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

### 📝 Couldn't be applied cleanly — apply manually (7)
- `models/user_model.py:71 (A02:2021) — no usable drop-in (see suggestion)`
- `.github/workflows/security-scan.yml:22-29 (A08:2021) — sketch, not auto-applied`
- `models/user_model.py:62-68 (A03:2021) — syntax error after fix, not auto-applied`
- `api_views/books.py:43-53 (A01:2021) — syntax error after fix, not auto-applied`
- `api_views/users.py:103-112 (A01:2021) — syntax error after fix, not auto-applied`
- `api_views/users.py:33-46 (A01:2021) — syntax error after fix, not auto-applied`
- `app.py:14 (A05:2021) — syntax error after fix, not auto-applied`

_These had no safe drop-in (no/sketchy code, bad line range, too large, would introduce a new issue, or wouldn't compile). The full suggested fix for each is in this run's numbered `security_findings/SECURITY-REVIEW.<n>.md` on this branch (and the `security-scan-report` artifact)._

---
> ⚠️ **Review every change. This PR is never auto-merged.** Merging is *your* decision to accept these AI suggestions; the merge commit re-runs the security gate and your own CI/tests.
