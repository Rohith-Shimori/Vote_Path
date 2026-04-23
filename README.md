# 🗳️ VotePath Pro+
### Intelligent Election Navigator

> **Hack2Skill Prompt Wars — Challenge 2**
> *Build an assistant that helps users understand the election process, timelines, and steps in an interactive and easy-to-follow way.*

---

## Design Philosophy

Most participants will build chatbots. VotePath Pro+ is not a chatbot.

It is a **structured, decision-driven civic navigation system** — built around deterministic logic, real data, and a user journey that is guided, not conversational.

> The system behaves like a civic GPS: it knows where you are, where you need to go, and tells you the exact steps to get there.

---

## What It Does

VotePath Pro+ guides any citizen through the complete election process for **19 countries** and **8 election types**, producing:

| Output | Description |
|---|---|
| **Readiness Score (0–100)** | Deterministic score based on age, registration, ID, and station awareness |
| **Status Classification** | NOT ELIGIBLE / NOT READY / IN PROGRESS / ALMOST READY / READY TO VOTE |
| **Risk Level** | CRITICAL / HIGH / MEDIUM / LOW — based on score vs. deadline proximity |
| **Election Timeline** | Visual: Today → Registration Deadline → Election Day → Results |
| **Readiness Checklist** | Exactly what you've done and what's still missing |
| **VotePath Journey** | Step-by-step simulation of your path to voting |
| **Personalized Insights** | Rule-based, contextual guidance — no AI involved |
| **Google Maps integration** | Direct link to find polling stations near you |
| **Google Calendar integration** | One-click to add Election Day to your calendar |

---

## Architecture

### Single-file, no framework, no backend

```
votepath-pro/
├── index.html          ← Entire application (HTML + CSS + JS)
├── data/
│   └── elections.json  ← Repository data source (hybrid pipeline)
└── README.md           ← This file
```

Total repository size: well under 1 MB.

---

### Core Engines

#### 1. User Intelligence Engine
Pure deterministic scoring — zero AI, zero randomness.

```
Score Breakdown:
  Age Eligibility    → 40 points (gate: below voting_age = score 0)
  Registered to vote → +30 (yes) / +10 (unsure) / +0 (no)
  Voter ID obtained  → +20 (yes) / +8 (applied) / +0 (no)
  Polling station    → +10 (known) / +0 (unknown)
                     ────────────────────────────
  Maximum            → 100 points

Status Labels:
  100        → READY TO VOTE  (green)
  76 – 95    → ALMOST READY  (blue)
  51 – 75    → IN PROGRESS   (amber)
  1  – 50    → NOT READY     (red)
  0          → NOT ELIGIBLE  (red)
```

#### 2. Risk Engine
Cross-references readiness score with days remaining until registration deadline and election.

```
score < 70  AND  daysToReg ≤ 7   → CRITICAL
score < 70  AND  daysToReg ≤ 30  → HIGH
score < 90  AND  daysToElection ≤ 60  → MEDIUM
otherwise                         → LOW
```

#### 3. Hybrid Data Pipeline
Demonstrates real-world resilience: tries to load live/updated data first, falls back gracefully.

```
Step 1: fetch('./data/elections.json')  ← Repository source (shown as "REPOSITORY DATA")
         ↓ if fails (timeout / CORS / missing)
Step 2: Use embedded ELECTION_DB object  ← (shown as "EMBEDDED DATA")
         ↓ in both cases
Step 3: normalizeData(raw) → canonical internal format
         ↓
Step 4: Engines receive clean, consistent data
```

The UI transparently shows users which data source is active via a status badge.

#### 4. Data Normalization Layer
All data — regardless of source — is normalized into a single canonical schema:

```json
{
  "registration_deadline": "YYYY-MM-DD",
  "election_date":         "YYYY-MM-DD",
  "result_date":           "YYYY-MM-DD",
  "poll_time":             "HH:MM AM – HH:MM PM",
  "authority":             "string",
  "id_requirements":       "string",
  "registration_url":      "https://...",
  "map_query":             "string (localized polling term)"
}
```

#### 5. Insight Engine
Generates actionable, personalized guidance from a rule matrix. No LLM involved.

Rules evaluate: score, risk level, days to deadlines, registration status, ID status, and station knowledge — producing up to 4 specific, non-generic insights.

#### 6. Journey Simulation Engine
Builds a visual, stateful step-path: each step is either `done`, `current`, or `upcoming` based on the user's actual profile.

#### 7. Google Services Integration

| Service | Integration |
|---|---|
| **Google Maps** | Deep link to search `{map_query} near {user_location}` |
| **Google Calendar** | Pre-filled event with election date, poll hours, ID requirements, and authority |

---

## How to Run

No build step. No npm. No server required.

```bash
git clone https://github.com/Rohith-Shimori/Vote_Path.git
cd Vote_Path
open index.html    # macOS
# or: double-click index.html in your file explorer
```

For the hybrid data pipeline to load `data/elections.json` as "REPOSITORY DATA", serve via a local server:

```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

Without a server (file:// protocol), the app falls back to embedded data automatically.

---

## Security

- No user data is stored — not in localStorage, not in cookies, not transmitted
- No API keys in the codebase
- External links use `rel="noopener noreferrer"`
- Input is validated before processing (age: numbers only, required fields checked)
- No `eval()`, no dynamic script injection

---

## Accessibility

- Semantic HTML5 structure throughout
- WCAG AA-compliant contrast ratios on all text
- Fully keyboard-navigable (Tab + Enter flow)
- All interactive custom elements have `role`, `tabindex`, and `aria-checked` attributes
- Skip-to-content link for screen reader users
- Responsive layout for mobile, tablet, and desktop
- Screen-reader friendly labels (`aria-label` on all inputs)

### 🎙️ Voice Accessible Navigator (V.A.N.)

A native, dependency-free voice assistant built with the Web Speech API:

- **Auto-offer on page load**: The app speaks a welcome message and instructs users to press **V** to start voice navigation
- **Keyboard shortcut**: Press **V** anytime (when not typing) to activate the voice wizard
- **Full wizard flow**: Country → Election Type → Age → Registration → Voter ID → Polling Station → Location
- **Readout**: After completion, the assistant reads the readiness score, status, and top insight aloud
- **Cancel anytime**: Say "cancel", "stop", or "quit" to exit
- **Browser support**: Chrome, Edge, Safari (voice input). Firefox (voice output only)
- **Zero external dependencies**: Uses only `speechSynthesis` and `SpeechRecognition` browser APIs

---

## Evaluation Alignment

| Criterion | Implementation |
|---|---|
| **Code Quality** | Clean separation of concerns: Data layer / Engine layer / Render layer. No framework bloat. |
| **Security** | No stored data, no exposed keys, validated inputs, safe external navigation |
| **Efficiency** | No build tools, no bundler, <1MB, instant load, CSS-only animations where possible |
| **Testing** | Graceful fallback on data fetch failure; input validation; edge cases handled (past dates, ineligible users) |
| **Accessibility** | Responsive, keyboard-navigable, high-contrast dark theme, semantic markup |
| **Google Services** | Google Maps (polling station search) + Google Calendar (election day event) |

---

## Countries Supported

🇺🇸 United States · 🇮🇳 India · 🇬🇧 United Kingdom · 🇦🇺 Australia · 🇨🇦 Canada · 🇩🇪 Germany · 🇫🇷 France · 🇧🇷 Brazil · 🇿🇦 South Africa · 🇯🇵 Japan · 🇵🇭 Philippines · 🇳🇬 Nigeria · 🇰🇪 Kenya · 🇮🇩 Indonesia · 🇲🇽 Mexico · 🇵🇰 Pakistan · 🇧🇩 Bangladesh · 🇬🇭 Ghana · 🌍 Other

---

## Election Types Supported

General Election · Presidential · State/Provincial · Local/Municipal · Referendum · Primary · Senate/Upper House · By-Election

---

## Assumptions

1. Election dates are approximate/representative for demonstration — users should verify with their official electoral commission
2. The hybrid pipeline's "live" source is the repo's `data/elections.json` — this is intentional for a static deployment
3. Readiness score is a civic guidance tool, not a legal determination of eligibility
4. Google Maps results depend on query quality and regional data availability
5. Voice navigation requires a Chromium-based browser (Chrome/Edge) for bidirectional interaction

---

*Built for Hack2Skill Prompt Wars Challenge 2 · No AI chatbot · Pure system design · 100% client-side*
