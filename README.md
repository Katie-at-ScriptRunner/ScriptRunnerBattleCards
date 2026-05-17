# ScriptRunner Battle Cards

A single-file sales and CS enablement tool for the ScriptRunner team. It turns competitive intelligence into a Pokémon-style card battle game — letting you explore objection handling, pricing arguments, and feature comparisons in a format that's fun to use and easy to remember. 

---

## What it is

Sales and CS reps choose one of three ScriptRunner products (SR for Jira, SR for Confluence, SR Connect), pick a competitor to battle, then engage in one of two modes:

- **Conversational Mode** — a branching dialogue that walks through real objection handling, pricing, positioning and use cases. Good for deep prep before a customer call.
- **Battle Arena Mode** — short, punchy move-by-move exchanges where HP bars drain and a winner is declared. Good for some quick wins!

Every opponent also has a **View Full Battle Card** modal packed with why-we-win points, objection handling, a pricing comparison table, use cases, and an automated score breakdown. This information has been gathered from various Confluence pages and Atlassian Marketplace and should be updated frequently.

The whole thing is a **single self-contained HTML file**. No server, no build step, no dependencies. Open it in a browser and it works. This should work with ScriptRunner Connect.

---

## The stack (plain language)

- **HTML/CSS** — structure and styling, all in one file
- **Vanilla JavaScript** — everything interactive: card building, battle sequences, scoring engine, modal. No frameworks, no libraries
- **Base64-encoded images** — logos are embedded directly in the file so it stays self-contained and portable
- **Google Fonts** — loaded from the web (Rajdhani, Share Tech Mono, Press Start 2P). Requires an internet connection for fonts, but the app functions without them.

---

## How the scoring works

Stats are not assigned manually. Every competitor and SR product has a `scoringText` field — a plain-text description of their capabilities, pricing model, support, and security posture. The scoring engine reads this text at page load and produces the numbers automatically.

### Step 1 — Signal extraction

The engine scans each `scoringText` for around 70 keyword patterns across five categories:

| Category | Example signals |
|---|---|
| **Ease of Use** | `no-code` (+25), `drag-and-drop` (+22), `barrier to entry low` (+20), `requires scripting` (-10) |
| **Flexibility** | `full Groovy scripting engine` (+28), `event listeners` (+14), `scheduled jobs` (+12), `preset conditions only` (-10) |
| **Security** | `ISO 27001` (+28), `SOC2` (+22), `Cloud Fortified` (+18), `data residency` (+16) |
| **Pricing** | `replaces multiple apps` (+28), `usage-based pricing` (+20), `per-user pricing` (-20), `per-task billing` (-22) |
| **Support** | `24/5 dedicated support` (+28), `40,000+ installations` (+22), `email support only` (-18) |

### Step 2 — Weighted scoring

Each matched signal contributes its weight to a raw score. A signal can fire multiple times (capped at 2x) to reward richer evidence. A product with many strong positive signals scores higher; one with several negative signals scores lower.

### Step 3 — Normalisation

Scores are normalised within each product group (SR + its competitors). The strongest scorer in a category gets around 95; the weakest gets around 40. This means scores are always relative — ScriptRunner consistently scores high because its `scoringText` contains the most positive signals.

### Step 4 — Explainability

Every score generates an explanation — a list of the signals that pushed it up and the ones that pushed it down. These appear in the **View Full Battle Card** modal under "How Scores Are Calculated" as green (positive) and red (negative) pills.

---

## Verified vs Unverified competitors

Competitors fall into two categories:

| Status | What it means |
|---|---|
| ✅ **Verified** | Content sourced from real official battle card documents. Safe to use in customer conversations. |
| ⚠️ **Being Updated** | Content based on industry knowledge. Needs review before use in the field. |

Unverified competitors show a yellow warning banner in the Battle Arena and at the top of the View Full Battle Card modal.

**Current verified competitors:**
- SR for Jira: Power Scripts, JMWE, JSU Automation Suite
- SR for Confluence: Automation for Confluence, Confluence CLI
- SR Connect: Exalate

**Currently unverified (pending review):**
- SR for Jira: Native Automation for Jira, Jira Workflow Toolbox
- SR for Confluence: Comala Workflows, Scroll Documents, Content Formatting Macros

---

## How to add a new competitor

Everything lives inside the single HTML file. Open it in a text editor and find the relevant section.

### 1. Add the competitor object to OPPONENTS

Find `const OPPONENTS = {` and locate the right product group (`jira`, `confluence`, or `connect`). Add a new object to the array:

```js
{
  id: 'mycompetitor',          // unique ID, no spaces, lowercase
  verified: false,             // set to true once it has been reviewed and verified
  name: 'My Competitor',       // display name
  cat: 'CATEGORY LABEL',       // short category shown on the card
  hp: 70,                      // starting HP (40–100, reflects relative strength)
  emoji: '🔧',                  // fallback if no logo
  desc: 'One-line description', // shown on the opponent card
  rarity: 'UNCOMMON',          // COMMON / UNCOMMON / RARE / LEGENDARY
  num: '#J06',                 // card number — increment from the last one
  color: {
    border: '#E53935',         // card border — any colour EXCEPT #FF0058 (SR Pink)
    bg: 'linear-gradient(160deg,#FFF3E0,#FFE0B2)',  // card background
    art: 'linear-gradient(135deg,#BF360C,#E64A19)', // art area background (if no logo)
  },
  stats: { ease:70, flex:50, sec:60, price:65, sup:55 }, // overwritten by scoring engine
  scoringText: `describe the product here in plain text. include features, pricing model,
    support type, security certifications, ease of use, barrier to entry. the more
    detail the more accurate the automated scoring will be.`,
  battleCard: {
    wins: [
      { title: 'Why we win point 1', body: 'Explanation...' },
      { title: 'Why we win point 2', body: 'Explanation...' },
    ],
    objections: [
      { q: 'Their objection.', a: 'SR response.' },
    ],
    usecases: [
      '🔧 Use case description',
    ],
    pricing: {
      srcNarrative: [`SR pricing narrative line 1.`, `Line 2.`],
      srcTable: [
        ['Users', 'SR for Jira (monthly/user)'],
        ['1-10', 'Free'],
        ['11-100', '$2.50'],
      ],
      oppNarrative: [`Competitor pricing description.`],
      oppTable: [
        ['Plan', 'Cost'],
        ['Paid', 'Check marketplace for current rates'],
      ],
      winReason: `One-line summary of why SR wins on pricing.`
    }
  }
}
```

### 2. Add battle sequences to allSeqs

Find `const allSeqs = {` and add a new entry matching the `id` you used above. Each topic (`price`, `integration`, `objection`, `positioning`, `usecase`) needs four steps:

```js
mycompetitor: {
  price: [
    { t:0,    announce:'PRICING BATTLE', lines:[[`>> MY COMPETITOR uses THEIR MOVE!`]], hp:opp.hp },
    { t:1800, lines:[[{win:`>> SR counters with OUR MOVE!`}]], hp:Math.round(opp.hp*.55) },
    { t:3600, lines:[[`>> MY COMPETITOR uses THEIR SECOND MOVE!`],[{win:`>> SR counters with OUR RESPONSE!`}]], hp:Math.round(opp.hp*.2) },
    { t:5400, lines:[[{ko:`MY COMPETITOR FAINTED!`}]], hp:0, ko:`Short knockout summary.` }
  ],
  integration: [ /* same format */ ],
  objection:   [ /* same format */ ],
  positioning: [ /* same format */ ],
  usecase:     [ /* same format */ ],
}
```

**Battle sequence rules:**
- Keep move names SHORT and ALL CAPS — this is a game, not a document
- `>> COMPETITOR uses MOVE NAME!` — the opponent attacks
- `>> SR counters with COUNTER MOVE!` — SR responds (wrapped in `{win: ...}`)
- `{ko: ...}` — the knockout line, also short
- `ko:` at the end of the last step — the one-line lesson that appears after the knockout
- Use `opp.hp` for the starting HP, then multiply down: `.55`, `.2`, `0`

### 3. Add a logo (optional)

Find `const LOGOS = {` near the top of the script and add the competitor ID with either a file path or base64 image:

```js
mycompetitor: './logos/mycompetitor.png',
// or
mycompetitor: 'data:image/png;base64,...',
```

If you leave it as `null`, the emoji fallback is used automatically.

### 4. Add scoringText to the product (SR side)

Each SR product also has a `scoringText` field that feeds the scoring engine. You should not need to change this unless SR's capabilities change materially. Find the product in `const PRODUCTS = {` and update the `scoringText` field accordingly.

---

## How to edit an existing competitor

### Updating battle card content

Find the competitor by its `id` in the `OPPONENTS` object. All content fields (`wins`, `objections`, `usecases`, `pricing`) live inside the `battleCard` property of that object.

### Updating battle sequences

Find the competitor's `id` inside `const allSeqs = {` and edit the relevant topic.

### Marking a competitor as verified

Find the competitor in `OPPONENTS` and change `verified: false` to `verified: true`. This removes the yellow warning banner and replaces the "Being Updated" badge with a green "Verified" tick.

### Updating the scoring text

Find the competitor in `OPPONENTS` and update the `scoringText` field. The scoring engine re-reads it every time the page loads — no other changes needed.

---

## How to add a logo

Find `const LOGOS = {` near the top of the script. The structure is:

```js
const LOGOS = {
  // SR Products
  jira:       'path/or/base64',
  confluence: 'path/or/base64',
  connect:    'path/or/base64',

  // Competitors
  exalate:    'path/or/base64',
  // etc.
};
```

Set the value to:
- A **relative path** (`'./logos/zapier.svg'`) if deploying with a folder of logo files
- An **absolute URL** (`'https://yourdomain.com/logos/zapier.png'`) if hosting the logos somewhere
- A **base64 data URI** (`'data:image/png;base64,...'`) to keep everything self-contained in one file

If a logo URL fails to load, the app automatically falls back to the emoji. Nothing breaks.

**Logo background:** The app automatically applies a white background behind any logo. The coloured gradient art background only shows when no logo is set.

---

## File structure

Everything is in one file: `scriptrunner-battlecards.html`

Inside the `<script>` tag, the code is organised in this order:

| Section | What it contains |
|---|---|
| `const LOGOS` | Logo file paths or base64 data URIs |
| `const PRODUCTS` | SR product definitions (Jira, Confluence, Connect) |
| `const STAT_DEFS` | Stat category definitions |
| `const OPPONENTS` | All competitor data including battle cards |
| `const SCORE_CONFIG` | Scoring signal patterns and weights |
| Scoring engine functions | `extractSignals`, `computeRaw`, `normalizeGroup`, etc. |
| `runScoringEngine()` | Runs at page load, writes scores back to all objects |
| `const allSeqs` | All battle sequences (Battle Arena mode) |
| UI functions | Card builders, modal, battle dialogue engine |
| Screen logic | Navigation between screens |

---

## Deployment

The file is self-contained and can be:
- Opened directly in any browser (no server needed)
- Hosted on any static file host (Netlify, S3, GitHub Pages, Confluence as an attachment)
- Deployed via ScriptRunner Connect itself

The only external dependencies are Google Fonts, which load from fonts.google.com. If the fonts fail to load (e.g. offline or restricted network), the app uses system fonts as a fallback and remains fully functional.

---

## Adding logos to missing competitors

The following competitors do not yet have logo files. Add them to `const LOGOS` when available:

| ID | Competitor | Notes |
|---|---|---|
| `nativeautomation` | Automation for Jira | Use Atlassian logo |
| `workflowtoolbox` | Jira Workflow Toolbox | Decadis brand |
| `autoconfluence` | Automation for Confluence | Use Atlassian logo |
| `comalaworkflows` | Comala Workflows | Comalatech brand |
| `scrolldocuments` | Scroll Documents | K15t brand |
| `contentmacros` | Content Formatting Macros | Appfire brand |

---

