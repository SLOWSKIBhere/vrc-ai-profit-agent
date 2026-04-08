# Security Policy

## Security Model

VRC AI Profit Agent is a fully client-side application. Understanding its security model is important before deploying it for real use.

### What is protected

| Asset | Protection |
|---|---|
| Login password | SHA-256 hashed with random 16-byte salt via Web Crypto API. Plaintext never stored anywhere. |
| Anthropic API key | AES-GCM 256-bit encrypted using a key derived from your password via PBKDF2 (100,000 iterations). Stored as an encrypted blob — never as plaintext. |
| Session | 2-hour idle auto-lock. All cryptographic keys wiped from memory on logout. |
| User inputs | Sanitised with `san()`, numeric-validated with `vN()`, HTML-escaped with `esc()` before any DOM render. |

### What is NOT protected

- **Data in localStorage is not end-to-end encrypted.** Daily records, khata entries, and settings are stored as JSON in `localStorage['vrc_v2']`. Someone with physical access to the browser's developer tools can read this data.
- **The Anthropic API key travels in HTTP request headers** when making AI calls. The encrypted storage protects it at rest, but it is visible in network traffic during an API call. Do not use this app on untrusted public networks if you are concerned about this.
- **There is no server-side authentication.** All security is browser-side. This is appropriate for a personal single-device tool. It is not appropriate as a multi-user hosted application without a proper backend.

### Recommended use

- Use on a personal, private device
- Enable your device's screen lock
- Use the JSON backup feature regularly (Settings → Export & Backup)
- Do not use on shared or public computers

### Reporting a vulnerability

If you find a security issue, please open a GitHub issue marked `[SECURITY]` or contact via LinkedIn.
