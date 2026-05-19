## Suggested security fixes — review, then merge if you agree
The security gate **blocked** [this scan](https://github.com/davidayomide/VAmPI/actions/runs/26073151089). This branch carries **AI-generated suggested fixes** for the findings, so you can review a concrete diff and decide. They are a starting point, not a guarantee — **review every line**.

> 🔒 **This PR is NEVER auto-merged.** Merging it is *your* explicit acceptance of these changes; the merge commit re-runs the security gate (and your own CI/tests).

### ✅ Suggested fixes on this branch (4)
- `models/user_model.py:57-63 (A03:2021)`
  - why: The get_user method constructs a raw SQL query by directly interpolating the username parameter into an f-string without any sanitisation or parameterisation. When vuln=1 (the default), any caller can supply an SQL injection payload as the username path parameter, causing the qu…
  - intentionally minimal; verify behaviour before merge.
- `config.py:11 (A05:2021)`
  - why: The JWT signing secret key is set to the static, trivially guessable string 'random'. Any attacker who knows (or guesses) this secret can forge valid JWT tokens for any username, including admin accounts, without needing to authenticate. Intentionally unchanged: the SECRET_KEY c…
  - ⚠️ **sensitive (protected path; touches SECRET_KEY)** — review this one extra carefully before accepting.
- `app.py:14 (A05:2021)`
  - why: The Flask application is started with debug=True in the production __main__ entry point. This enables the Werkzeug interactive debugger, which exposes a PIN-protected Python REPL accessible from the browser. If the PIN is leaked or bypassed, an attacker gains remote code executi…
  - ⚠️ **sensitive (manual-review category (remote code execution))** — review this one extra carefully before accepting.
- `api_views/main.py:11-13 (A03:2021)`
  - why: The basic() function constructs a JSON response by direct string concatenation of the vuln variable into the response body without using json.dumps or jsonify. While vuln is an integer derived from os.getenv, the pattern of building JSON through string formatting is unsafe and d…
  - intentionally minimal; verify behaviour before merge.

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
- `api_views/books.py:43-53 (A01:2021) — no usable drop-in (see suggestion)`
- `api_views/users.py:113-120 (A01:2021) — no usable drop-in (see suggestion)`
- `api_views/users.py:36-50 (A07:2021) — no usable drop-in (see suggestion)`
- `models/user_model.py:26 (A02:2021) — no usable drop-in (see suggestion)`
- `api_views/users.py:23-25 (A09:2021) — no usable drop-in (see suggestion)`
- `api_views/users.py:11-14 (A03:2021) — syntax error after fix, not auto-applied`

_These had no safe drop-in (no/sketchy code, bad line range, too large, would introduce a new issue, or wouldn't compile). The full suggested fix for each is in this run's numbered `security_findings/SECURITY-REVIEW.<n>.md` on this branch (and the `security-scan-report` artifact)._

---
> ⚠️ **Review every change. This PR is never auto-merged.** Merging is *your* decision to accept these AI suggestions; the merge commit re-runs the security gate and your own CI/tests.

### ⚠️ Automated self-check advisories
These did **not** block this PR — *you* are the reviewer. Weigh them:

- ⚠️ ruff/bandit flagged a NEW lint/security issue in a suggested fix — review carefully
