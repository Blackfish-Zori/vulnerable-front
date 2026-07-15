# Vulnerable Front App

A deliberately insecure application, scaffolded with `create-react-app`, built
for exercising **SAST** (static analysis). It is **not** a production template — do not deploy it
publicly or reuse these patterns.

## What's inside

```
vulnerable-front/
├── src/
│   ├── config.js                  # hardcoded secrets (SAST secret-scanning)
│   ├── App.js                     # wires up the panels below
│   └── components/
│       ├── XssDemo.js             # reflected/DOM XSS, eval()
│       ├── InsecureStorage.js     # localStorage secrets, weak crypto/random, cookie flags
│       └── RedirectAndFetch.js    # open redirect, client-side SSRF-style fetch
└── package.json                   # includes known-vulnerable dependency versions
```

## Running the frontend (SAST)

```bash
cd vulnerable-test-app
npm install
npm start        # http://localhost:3000
```

Point your SAST tool at the `src/` folder (and `package.json`/`package-lock.json`
for software-composition-analysis / dependency scanning). Point your DAST tool
at `http://localhost:3000` once it's running.

## Vulnerabilities included

### Frontend (React / `src/`)
| Vulnerability | Location | Notes |
|---|---|---|
| Hardcoded secrets (API keys, AWS creds, DB connection string, JWT secret) | `src/config.js` | Classic SAST secret-detection targets |
| Reflected XSS | `XssDemo.js` (`dangerouslySetInnerHTML`) | User input rendered as raw HTML |
| DOM-based XSS | `XssDemo.js` (`innerHTML` from `location.search`) | Sink fed by untrusted URL param |
| Use of `eval()` on user input | `XssDemo.js` | Code injection sink |
| Sensitive data in `localStorage` | `InsecureStorage.js` | Password, credit card, session token stored unencrypted |
| Insecure cookie (no `Secure`/`HttpOnly`/`SameSite`) | `InsecureStorage.js` | |
| Weak randomness (`Math.random()` for tokens) | `InsecureStorage.js` | Predictable session tokens |
| Weak/home-grown "hashing" | `InsecureStorage.js` | Not a real cryptographic hash |
| Open redirect | `RedirectAndFetch.js` | Unvalidated `window.location.href` |
| Client-side SSRF-style fetch | `RedirectAndFetch.js` | Fetches attacker-supplied URL with no allow-list |
| Vulnerable dependency versions | `package.json` | `lodash@4.17.15`, `axios@0.21.0`, `jsonwebtoken@8.5.1`, `js-yaml@3.13.0` — all have known CVEs, good for SCA/dependency-audit tools |

## Suggested manual test cases

- `XssDemo`: submit `<img src=x onerror=alert(1)>` in the comment box; visit
  `http://localhost:3000/?name=<script>alert(1)</script>`.
- `InsecureStorage`: log in, then inspect DevTools → Application → Local
  Storage / Cookies.
- `RedirectAndFetch`: try `javascript:alert(1)` or a non-allow-listed internal
  URL.
