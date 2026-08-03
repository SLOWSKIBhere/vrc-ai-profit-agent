# Security Policy

## Security Model

VRC AI Profit Agent is a fully client-side application. Understanding its security model is important before deploying it for real use.

### What is protected

| Asset | Protection |
|---|---|
| Login password | PBKDF2-SHA-256 (250,000 iterations, random 16-byte salt). A known verifier string is AES-GCM encrypted with the derived key and stored. At login, decryption success/failure is the auth check. Plaintext password never stored. |
| OpenRouter API key | AES-GCM 256-bit encrypted using a key derived from your password via PBKDF2 (250,000 iterations). Stored as an encrypted blob (`vrc_sec`) — never as plaintext. Key type is OpenRouter, not Anthropic. |
| Session | 2-hour idle auto-lock. All cryptographic keys wiped from memory on logout. |
| User inputs | Sanitised with `san()`, numeric-validated with `vN()`, HTML-escaped with `esc()` before any DOM render. |

Legitimate legacy authentication data that used a SHA-256 password hash remains supported for one-time migration after a successful login. Legacy 100,000-iteration API-key encryption is also supported during that migration; new and rotated credentials use 250,000 iterations.

### What is NOT protected

- **Data in localStorage is not end-to-end encrypted.** Daily records, khata entries, and settings are stored as JSON in `localStorage['vrc_v2']`. Someone with physical access to the browser's developer tools can read this data.
- **The OpenRouter API key travels in HTTP request headers** when making AI calls. The encrypted storage protects it at rest, but it is visible in network traffic during an API call. Do not use this app on untrusted public networks if you are concerned about this.
- **There is no server-side authentication.** All security is browser-side. This is appropriate for a personal single-device tool. It is not appropriate as a multi-user hosted application without a proper backend.

- **`script-src 'unsafe-inline'` in the Content-Security-Policy** — the entire application
  is an inline `<script>` block inside a single HTML file, which requires `'unsafe-inline'`.
  This means CSP provides no protection against injected inline scripts (e.g., from a
  malicious browser extension). The Chart.js CDN import has a `crossorigin` + `integrity`
  attribute that does provide SRI verification. Mitigation: do not install untrusted browser
  extensions; verify the app is loaded from a trusted source.

- **`connect-src` formerly permitted `https://api.anthropic.com`** — this host was never
  called by the application; all AI requests go to `https://openrouter.ai`. The unused,
  wider-than-necessary permission was removed in [P1-I1]. Restore it only if a future
  version switches to the direct Anthropic API, and update the documentation at that time.

### Recommended use

- Use on a personal, private device
- Enable your device's screen lock
- Use the JSON backup feature regularly (Settings → Export & Backup)
- Do not use on shared or public computers

### Reporting a vulnerability

If you find a security issue, please open a GitHub issue marked `[SECURITY]` or contact via LinkedIn.
