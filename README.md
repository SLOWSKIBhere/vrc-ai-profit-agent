# VRC AI Profit Agent

**A low level agentic AI system built for a real person solving a real problem.**

A browser-based 4-agent AI pipeline that helps small shop owners in India track daily UPI and cash income, manage expenses and credit, and receive Claude-powered business intelligence — all without a backend, server, or installation.

> Built as a first agentic AI project by a 10th grade student at Central Bucks South, PA.  
> Inspired by observing a shop owner manually reconciling UPI notifications against a paper ledger every evening during a trip to Chintalapudi, Andhra Pradesh, India.

---

## What This Is

Most AI demos are built for default users in controlled environments.

This was built for a specific person — a cloth shop owner in rural Andhra Pradesh — with a specific payment context (UPI + cash), a specific language environment (English + Hindi), and a specific threat model (shared family device, no guaranteed private access).

Every technical decision in this project came from that reality.

---

## Live Demo

Open `index.html` in any modern browser. No install. No backend. No dependencies to install.

On first open, a 3-step setup wizard runs to configure your shop name, password, and daily revenue target.

**Demo mode:** Click `🎭 Load Realistic Demo Data` on the Entry tab to populate with realistic Andhra Pradesh cloth shop data.

---

## Features

### Core
- **UPI message parser** — reads English and Hindi payment notifications  
  `"Received Rs 850 from Lakshmi via PhonePe"` → ₹850  
  `"panch sau rupaye prapt hue"` → ₹500  
  `"UPI/CR/1200/GooglePay"` → ₹1200
- **Cash income tracking** with daily target progress
- **Expense tracker** with 7 categories (Wholesale Fabric, Staff Wages, Electricity, etc.)
- **Khata (credit) book** — track credit given to customers, mark as paid
- **Festival Mode** — toggles revenue benchmarks and target bars for high-season days

### Analytics
- **Today tab** — real-time profit/loss badge, margin %, expense ratio warnings
- **Week tab** — Chart.js bar chart of 7-day profit history, best/worst day
- **Simulation** — realistic normal week and festival week simulator based on actual market data

### AI Layer — 4-Agent Pipeline

| Agent | Role | What it does |
|---|---|---|
| Agent 1 | UPI Parser | Loads and validates the day's data |
| Agent 2 | Data Validator | Catches anomalies before AI sees them (revenue 3× average? expense ratio >80%?) |
| Agent 3 | Trend Analyst | Rule-based stats across last 14 days |
| Agent 4 | Claude AI | Generates daily or weekly business intelligence using real Indian market benchmarks |

**Prompt engineering:** The AI prompt includes actual Andhra Pradesh cloth shop benchmarks — normal day ₹2,500–6,000, festival day ₹12,000–45,000, expected margins 25–40% — so Claude's advice is calibrated to the actual context, not generic business advice.

### Sharing
- **WhatsApp daily report** — formatted profit summary, one tap to share

### Data
- **CSV export** — for Excel / Google Sheets
- **JSON backup + restore** — full data backup with smart merge on restore

---

## Security Architecture

This was designed around a real threat model: a shared family device with no guaranteed private access, and an OpenRouter API key with real monetary value.

| Layer | Implementation |
|---|---|
| Password verification | PBKDF2-SHA-256 (250,000 iterations, 16-byte random salt) derives an AES-GCM key that decrypts a known verifier; plaintext is never stored |
| Key derivation | PBKDF2 iteration values are bounded from 100,000 to 1,000,000; invalid stored values use the 250,000-iteration default |
| API key encryption | AES-GCM (256-bit) — the OpenRouter key never touches localStorage in plaintext |
| Session management | 2-hour idle auto-lock, 15-minute warning banner |
| Input sanitisation | `san()`, `esc()`, `vN()` on all user inputs |
| XSS prevention | All dynamic HTML escaped before render |

Current setups persist neither the plaintext password nor a fast password hash. Legitimate legacy SHA-256 password data remains supported for one-time migration after a successful login.

All cryptography uses the browser's native **Web Crypto API** — zero external dependencies.

```
User password
     │
     ▼  PBKDF2 (250,000 iterations, random 16-byte salt, SHA-256)
 AES-256-GCM key  ←── memory only (sessionKey), never written to disk
     ├── decrypt known auth verifier → localStorage['vrc_setup']
     └── encrypt OpenRouter API key  → localStorage['vrc_sec']
```

---

## Version History

This project was built iteratively with a formal audit between each phase — treating a student prototype like a production system.

| Version | What changed |
|---|---|
| `v1` — Initial prototype | UPI parser, daily entry, basic AI call, localStorage |
| `v2` — Deployment audit | Full readiness report written (scored 62/100), gaps identified |
| `v3` — Phase 1 fixes | Removed hardcoded password hint, fixed duplicate UPI bug, added real 4-agent pipeline, JSON backup/restore, 3-step entry guide |
| `v4` — Phase 2 security | First-run setup wizard, PBKDF2-SHA-256 password verification, AES-GCM API key encryption, 2-hour idle auto-lock, AI quota tracker (3 calls/day), fetchWithRetry with friendly error handling |
| `v5` — Phase 3 resilience | Day-of-week patterns, same-day AI response cache, login throttling, storage warnings, and empty-save protection |
| `v6` — Current | OpenRouter Claude pipeline, PBKDF2-derived AES-GCM password verifier, CSP/SRI hardening, atomic backup validation, India-local dates, cache correctness, and overlapping-run protection |

---

## Technical Stack

```
Frontend      Vanilla JS (ES2022) · HTML5 · CSS3
Crypto        Web Crypto API — PBKDF2, AES-GCM, SHA-256
AI            OpenRouter Chat Completions API (anthropic/claude-sonnet-4)
Charts        Chart.js 4.4.0
Storage       localStorage (client-side only, no backend)
Size          Single HTML file · no build step · zero npm installs
```

---

## How to Use

### Option A — Just open it
Download `index.html` and open it in Chrome, Firefox, or Safari. That's it.

### Option B — Clone the repo
```bash
git clone https://github.com/SLOWSKIBhere/vrc-ai-profit-agent.git
cd vrc-ai-profit-agent
# Open index.html in your browser
```

### Setup (first open)
1. Complete the 3-step wizard — shop name, password, daily target
2. Go to **⚙️ More → AI Agent Configuration** to add your OpenRouter API key
3. Start entering daily data on the **📝 Entry** tab

**Developer manual test — deferred password migration:** Create a legacy account record containing `pwdHash`, fill `localStorage` close enough to quota that `saveSetup(migrated)` throws, then log in. Login should still succeed and the agent bar should show “⚠️ Security upgrade pending. Go to ⚙️ Settings → Change Password to complete it.” When storage has capacity and migration succeeds, no warning should appear. Clear the test data afterward.

---

## Known Limitations

- **Not tested with live shop data across multiple real weeks** — this is the honest gap. Demo data and simulated entries were used for development. Real-world pilot testing is the active next step.
- **localStorage only** — data lives on the device and browser used. JSON backup is available but requires manual export.
- **Single-device** — no cloud sync across devices.
- **AI calls route through OpenRouter** — the app calls `https://openrouter.ai/api/v1/chat/completions`
  using an OpenRouter API key. The key is AES-GCM encrypted in storage but travels in the
  `Authorization: Bearer` header on each request. Do not use on untrusted public networks.
  The model is `anthropic/claude-sonnet-4` via OpenRouter — not a direct Anthropic API call.

---

## What I Learned Building This

**Security architecture from a real threat model.** I didn't plan to implement PBKDF2 and AES-GCM. I planned to add a login screen. Thinking seriously about who would use this — a shop owner, shared phone, public WiFi at a local market — forced every security decision that followed.

**Prompt engineering requires domain knowledge.** Dropping a data object into a generic AI prompt produces generic advice. Baking in actual regional market benchmarks, expense ratios, and seasonal patterns turned the same Claude API call into genuinely useful, contextually calibrated output.

**The feature that gets used is not the feature you're most proud of.** The WhatsApp daily report button — the simplest thing in the entire app — is the one that actually fits how information moves in that context. That lesson is worth more than any technical implementation.

**Constraints are a design tool.** No time for a backend → cleaner single-file architecture. No real test data → better simulation logic and honesty about limitations. Shared device threat model → proper cryptography.

---

## Roadmap

- [x] Phase 3 — Day-of-week patterns, AI caching, login throttling, and storage safeguards
- [ ] Phase 4 — Real-world pilot with 1 shop owner over 4+ weeks
- [ ] Telugu language support for labels and AI prompts  
- [ ] Multi-shop support (multiple localStorage namespaces or Supabase backend)
- [x] First-run setup personalisation (configured shop name used throughout)
- [ ] Automated festival calendar (Ugadi, Sankranti, Dussehra, Diwali) for AP region

---

## Deployment Readiness Audit

Before writing Phase 1 fixes, I wrote a full deployment readiness report treating this prototype as if it were going to a real user. Scored across 7 dimensions:

| Dimension | Score |
|---|---|
| UX Design Quality | 78% |
| Feature Completeness | 68% |
| AI / Agentic Depth | 42% → improved in v4 |
| Data Resilience | 30% → improved in v3/v4 |
| Security | 28% → improved in v4 |
| Testing Coverage | 15% |
| Real-World Readiness | 45% |
| **Overall** | **62 / 100** |

The audit document is what drove the v3 and v4 builds. Writing it before coding was the most useful thing I did on this project.

---

## License

MIT — use it, learn from it, build on it.

---

## Author

**Kowshik Jallipalli**  
10th Grade · Central Bucks South · Chalfont, PA  
[LinkedIn](https://www.linkedin.com/in/kowshik-jallipalli-a7b301229)

*First agentic AI project. More coming.*
