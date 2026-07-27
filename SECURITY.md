# Security Policy

## Security Model

VRC AI Profit Agent is a fully client-side application. Understanding its security model is important before deploying it for real use.

### What is protected

| Asset | Protection |
|---|---|
| Login password | Current setups use PBKDF2-SHA-256 (250,000 iterations with a random 16-byte salt) to derive an AES-GCM key that decrypts a known verifier. Neither plaintext nor a fast password hash is persisted for current setups. |
| OpenRouter API key | AES-GCM 256-bit encrypted using the PBKDF2-derived password key. Stored as an encrypted blob — never as plaintext. |
| Session | 2-hour idle auto-lock. All cryptographic keys wiped from memory on logout. |
| User inputs | Sanitised with `san()`, numeric-validated with `vN()`, HTML-escaped with `esc()` before any DOM render. |

Legitimate legacy authentication data that used a SHA-256 password hash remains supported for one-time migration after a successful login. Legacy 100,000-iteration API-key encryption is also supported during that migration; new and rotated credentials use 250,000 iterations.

### What is NOT protected

- **Data in localStorage is not end-to-end encrypted.** Daily records, khata entries, and settings are stored as JSON in `localStorage['vrc_v2']`. Someone with physical access to the browser's developer tools can read this data.
- **The OpenRouter API key travels in HTTP request headers** when making AI calls. The encrypted storage protects it at rest, but it is visible in network traffic during an API call. Do not use this app on untrusted public networks if you are concerned about this.
- **There is no server-side authentication.** All security is browser-side. This is appropriate for a personal single-device tool. It is not appropriate as a multi-user hosted application without a proper backend.

### Recommended use

- Use on a personal, private device
- Enable your device's screen lock
- Use the JSON backup feature regularly (Settings → Export & Backup)
- Do not use on shared or public computers

### Reporting a vulnerability

If you find a security issue, please open a GitHub issue marked `[SECURITY]` or contact via LinkedIn.
