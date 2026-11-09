# 🏈 NFL Analyzer (v1.0)

[![Version](https://img.shields.io/badge/version-1.0.0-emerald.svg)](https://github.com)
[![Framework](https://img.shields.io/badge/Vue.js-3.x-4fc08d.svg)](https://vuejs.org/)
[![Styling](https://img.shields.io/badge/Tailwind-CSS-38bdf8.svg)](https://tailwindcss.com/)
[![Database](https://img.shields.io/badge/Google%20Cloud-Firestore-ffca28.svg)](https://firebase.google.com/)
[![Data](https://img.shields.io/badge/Google%20Sheets-API%20%2F%20CSV-34a853.svg)](https://www.google.com/sheets/about/)
[![Live Feed](https://img.shields.io/badge/ESPN-NFL%20Scoreboard%20API-cc0000.svg)](https://www.espn.com/nfl/)

An institutional-grade, real-time NFL analytical consensus and matchup handicapping dashboard. The web app ingests multi-model projections across two live Google Sheets (**Spread & Moneyline** and **Total / Over-Under**), evaluates them against real-time ESPN market lines using tiered consensus algorithms, and persists verified performance records automatically to **Google Cloud Firestore**.

---

## 🌟 Key Features

* **Multi-Market Consensus Engine**: Independent quantitative evaluations for **Point Spread**, **Moneyline (ML)**, and **Game Total (Over/Under)** for every matchup.
* **Tiered Consensus & Contrarian Protection**:
  * **Spread**: 20 models organized across 3 tiers with automated Tier 3 contrarian downgrade filters.
  * **Moneyline**: 64 models organized across 3 tiers, calculating expected value ($\text{EV}\%$) and detecting Tier 1/Tier 2 outright winner splits.
  * **Totals**: 15 models across 3 tiers benchmarking model projections against live market totals.
* **Live ESPN Integration**: Real-time scoreboard with live score updates, period/game clock, down-and-distance situational tracking, and live in-game cover probability calculations.
* **Permanent Cloud Ledger**: Automated game grading ($\text{W} / \text{L} / \text{P}$) persisted in real-time to Google Cloud Firestore with season, month, week, and market filters.
* **Advanced EPA & Efficiency Deep-Dive**: 32-team database featuring Offensive/Defensive EPA per play, Net EPA, Pass/Rush EPA splits, turnover margin, and head-to-head records.
* **Zero-Build Architecture**: Runs directly in any modern browser via CDN-hosted Vue 3, Tailwind CSS, and the Firebase Web SDK—ready for immediate hosting on **GitHub Pages**.

---

## 🧠 Model Tiers & Decision Rules

### 1. 🏈 Point Spread Engine (20 Models)

The spread engine calculates $\Delta = |\text{Model Consensus Spread} - \text{Live Market Line}|$.

| Tier | Weight / Purpose | Included Models |
| :--- | :--- | :--- |
| **Tier 1 (Core)** | Primary driver (5 models) | `lineeff`, `linecong`, `lineclean`, `lineffw`, `linel2` |
| **Tier 2 (Secondary)** | Supporting consensus (7 models) | `linecoff`, `linelog`, `linepi`, `linedonchess`, `linepyth`, `linel1`, `linemed` |
| **Tier 3 (Check)** | Contrarian checks (8 models) | `linedok`, `linecurry`, `lineavg`, `linemass`, `lineppg`, `lineespn`, `linesaggm`, `linesag` |

#### Spread Decision Rules
* **Strong Bet**: $\Delta \ge 2.5\text{ pts}$ with Tier 1 consensus.
* **Lean**: $0.0 \le \Delta \le 2.0\text{ pts}$.
* **Tier 3 Contrarian Filter**: If Tier 3 models strongly oppose the Tier 1 direction, a **Strong Bet** is automatically downgraded to a **Lean** with an in-app visual indicator (`⚠️ T3 Contrarian`).

---

### 2. 💰 Moneyline (ML) Engine (64 Models)

Evaluates projected victory margin, model win probability, and market-implied win probability to find positive expected value ($\text{EV}\%$).

| Tier | Quantity | Included Models |
| :--- | :---: | :--- |
| **Tier 1 (Core)** | 21 Models | `linefeng`, `lineherbert`, `linegt`, `linesagr`, `lineca`, `linemass`, `linemed`, `linenfelo`, `lineavg`, `linemore`, `lineespn`, `linedonchess`, `linesag`, `linestjohn`, `lineteamrank`, `linecovers`, `linesaggm`, `linebihl`, `lineffw`, `linepib`, `lineform` |
| **Tier 2 (Secondary)** | 21 Models | `linepi`, `linecong`, `linesagp`, `lineARGH`, `linepim`, `linedok`, `linecoff`, `linedoi`, `linecurry`, `linedunk`, `lineash`, `lineesp2`, `linetalis`, `linesage`, `linepz`, `linepig`, `linel2`, `lineppg`, `linegametime`, `linekerns`, `linepugh` |
| **Tier 3 (Check)** | 22 Models | `linel1`, `linejohns`, `linel2to`, `lineeff`, `linepyth`, `lineesp`, `linejens`, `lineelo`, `linefox`, `lineclean`, `linelog`, `linerwp`, `lineround`, `linehanson`, `lineexcel`, `lineshark`, `linel2h`, `linedwig`, `linegrok`, `lineturner`, `lineexcel2`, `linejens2` |

#### Moneyline Decision Rules
* **Strong Bet**: Tier 1 Consensus projecting $|\text{Margin}| \ge 6.0\text{ pts}$ (with no outright winner disagreement between Tier 1 and Tier 2).
* **Lean**: Models projecting $0.0 \le |\text{Margin}| \le 6.0\text{ pts}$, **or** where **Tier 1 and Tier 2 split on the outright winner** (`⚠️ T1 & T2 Winner Split`).

---

### 3. 📊 Total (Over/Under) Engine (15 Models)

Evaluates projected game points against the live ESPN total line ($\Delta = |\text{Model Total} - \text{Live Total Line}|$).

| Tier | Weight / Purpose | Included Models |
| :--- | :--- | :--- |
| **Tier 1 (Core)** | Primary consensus (4 models) | `totdonchess`, `totturing`, `totffw`, `totpugh` |
| **Tier 2 (Secondary)** | Secondary benchmark (6 models) | `totavg`, `totmed`, `totmass`, `totclean`, `totargh`, `totpirate` |
| **Tier 3 (Check)** | Contrarian check (5 models) | `totsag`, `totrwp`, `totashby`, `totcurry`, `totkerns` |

#### Total Decision Rules
* **Strong Bet (`STRONG OVER` / `STRONG UNDER`)**: $2.5 \le \Delta \le 4.5\text{ pts}$.
* **Lean (`OVER LEAN` / `UNDER LEAN`)**: $\Delta \le 2.0\text{ pts}$ (or when downgraded by $\ge 60\%$ opposing Tier 3 consensus).

---

## 🔌 Data Sources & Google Sheets Setup

The app dynamically fetches public CSV endpoints from Google Sheets via the Google Visualization API (`/gviz/tq?tqx=out:csv`).

| Data Feed | Google Sheet ID | Description |
| :--- | :--- | :--- |
| **Spread & Moneyline** | `1ElxfamOBG5IBhK1YdsuHoeV17ayQCJ1JZdn50c_9WbI` | Contains matchup data, live spread lines, and the 20 spread / 64 ML model columns. |
| **Total (O/U)** | `1-cFvXE-svW8NbNtCGvy1HtlyI75qCLkw5RvojqCzU9Q` | Contains projected game totals and the 15 total model columns. |

> **Column Compatibility**: The built-in alias parser normalizes case, spaces, and prefixes (e.g., `lineeff`, `eff`, `line_eff`, and `line eff` resolve to the same metric).

---

## 🗄️ Cloud Persistence (Firestore)

Historical plays and finalized game results are automatically synced to Google Cloud Firestore under:

```text
/artifacts/{appId}/public/data/settled_games/{docId}
```

* **Auto-Grading**: When an ESPN game status updates to `FINAL`, the app grades the spread, moneyline, and total plays as `W` (Win), `L` (Loss), or `P` (Push).
* **Multi-Device Synchronization**: Real-time snapshot listeners ensure ledger records update across all open instances without requiring manual page refreshes.

---

## 🚀 Quick Start & Deployment

### Local Development
Because this is a client-side single-page application, no Node.js build step is required:

1. **Clone the repository**:
   ```bash
   git clone https://github.com/your-username/nfl-analyzer.git
   cd nfl-analyzer
   ```
2. **Open in browser**:
   * Double-click `index.html` to open it directly in Chrome, Firefox, Safari, or Edge.
   * Or run a local HTTP server:
     ```bash
     python -m http.server 8000
     # Navigate to http://localhost:8000
     ```

### Deploying to GitHub Pages
1. Push your repository to GitHub:
   ```bash
   git add .
   git commit -m "feat: release NFL Analyzer v1.0"
   git push origin main
   ```
2. In your GitHub repository:
   * Go to **Settings** > **Pages**.
   * Under **Build and deployment** > **Source**, select `Deploy from a branch`.
   * Set the branch to `main` and the folder to `/ (root)`.
   * Click **Save**. Your site will be live at `https://your-username.github.io/nfl-analyzer/`.

---

## 🛠️ Tech Stack

* **Frontend**: [Vue.js 3](https://vuejs.org/) (Composition API via CDN)
* **Styling**: [Tailwind CSS](https://tailwindcss.com/) (Just-In-Time CDN)
* **CSV Parsing**: [PapaParse 5.4](https://www.papaparse.com/)
* **Database / Backend**: [Firebase JS SDK v11](https://firebase.google.com/docs/web/setup) (Authentication & Cloud Firestore)
* **Live Sports API**: ESPN Public NFL Scoreboard Endpoints

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.