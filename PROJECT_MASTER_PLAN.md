# 📊 Project Master Plan: European Dividend Portfolio Tracker

> **Your Excel file, transformed into a living web application.**

---

## 🔍 Excel File Analysis — What You Have Today

Your spreadsheet (`UOS - Carteira de Dividendos EUROPA`) is a **personal investment portfolio tracker** with 11 sheets:

| Sheet | What it contains |
|---|---|
| **Painel** | Dashboard for the main portfolio (EPA:RUI focus). KPIs: current value, positions, cash, unrealized gains, dividends. |
| **Painel VINCI** | Separate dashboard for the Vinci/PEE portfolio (work savings plan). |
| **Ações** | Holdings table: 11 stocks with ticker, company, quantity, average price, current price, currency, daily variation, total value, weight %. |
| **Ações Vinci** | Holdings for the PEE plan (Vinci employer plan): 4 funds including Epargne Actions, Amundi bonds, etc. |
| **Histórico** | Transaction log: 32 trades with date (Excel serial), ticker, action (Compra/Venda/Dividendos/Depósito), quantity, price, total. |
| **Histórico - PEE** | Transaction history for the Vinci PEE (empty/minimal). |
| **Prices** | Manual price lookup table for 3 PEE funds (no market API). |
| **Alocação** | Buy-zone analysis: required yield per country, current yield, buy-zone price target, distance from buy zone, next dividend forecast. |
| **Analise dos dividendos** | Dividend breakdown by company and year (2023–2026), including companies that left the portfolio. |
| **Fundo Small cap** | Data for the small-cap fund. |
| **Testes** | Scratch/test area. |

### Key Data Points Extracted

**Two portfolios co-exist:**
- **Main portfolio** ("Renda Gringa"): 11 European stocks traded on Euronext Paris (EPA:), Lisbon (ELI:), and via ISIN. Total value ~€3,780.
- **PEE portfolio** (Vinci employer plan): 4 Amundi funds. Total value ~€6,499.

**Transaction types:** `Compra` (buy), `Venda` (sell), `Dividendos` (dividend received), `Depósito` (cash deposit), `Retirada` (withdrawal).

**Core calculations the app needs to replicate:**
- Current portfolio value = sum(quantity × current price)
- Unrealized gain = current value − invested value
- Dividend yield = annual dividend / current price
- Buy-zone threshold = expected annual dividend / required yield rate (e.g., 6.5% for French stocks)
- Portfolio weight = stock value / total portfolio value

---

## 1. 🎯 Project Overview

### What the app does (in simple terms)

Think of it like a **live GPS for your investments** — instead of opening Excel every day, you open a website that:

1. Shows you all your stocks and funds in real time
2. Tells you if a stock is in the "buy zone" (good price to buy more)
3. Logs every trade, deposit, and dividend you receive
4. Shows charts of how your portfolio grew over time
5. Summarizes your dividend income by month and year

### Main Goals

- Replace the manual Excel workflow with an automated, visual web app
- Track **two separate portfolios** (personal brokerage + Vinci PEE)
- Never lose transaction history (durable database instead of a fragile Excel file)
- Provide a clean dashboard accessible from any device (phone, laptop)

### Target Users

- **You** — the sole user, at first. A single-user personal finance tracker.
- Later, potentially shared with your investment club (Clube de Valor)

---

## 2. 🛠️ Recommended Tech Stack

> "Choosing a tech stack is like choosing your construction materials. For a house you'll live in alone, you don't need industrial concrete — but you do want something solid enough to expand later."

### The Stack

| Layer | Technology | Why |
|---|---|---|
| **Backend** | Python + FastAPI | Python is the most beginner-friendly language. FastAPI is modern, fast, and has automatic API documentation. |
| **Frontend** | React (via Vite) | The most in-demand frontend framework. Claude can generate excellent React code. Clean component model. |
| **Database** | SQLite → PostgreSQL | SQLite requires zero setup — it's a single file. When you're ready to deploy to the cloud, you migrate to PostgreSQL. |
| **ORM** | SQLAlchemy | Lets you talk to the database in Python instead of raw SQL. Easier to learn. |
| **Styling** | TailwindCSS | Utility-first CSS. You write styles directly in HTML/JSX. No separate CSS files to manage. |
| **Charts** | Recharts | React-native chart library, beginner-friendly, beautiful defaults. |
| **Package manager** | uv | Replaces pip + venv in one tool. 10–100x faster than pip. Manages Python version, virtual environment, and dependencies through a single `pyproject.toml`. No `requirements.txt`, no manual `venv` activation. |
| **Auth** | None (MVP) → HTTP Basic | Start with no auth (local only). Add simple password protection before deploying. |
| **Deployment** | Railway or Render (free tier) | One-click deploys. No server management needed. |

### Why this stack for a beginner?

- **Python** is English-like and forgiving. You probably already encountered it in engineering.
- **uv** handles everything in one command — installing Python, creating the environment, adding packages. You never type `pip install` or `source venv/bin/activate` again.
- **FastAPI** auto-generates a visual API explorer at `/docs` — you can test your backend without writing any frontend code.
- **SQLite** is literally a single `.db` file. You can open it, inspect it, back it up by copying one file.
- **React + Vite** gets you running in under 5 minutes with `npm create vite@latest`.
- **Recharts** makes charts with ~10 lines of code.

### Civil engineering analogy

Your app is like a building:
- **FastAPI** = the structural frame (holds everything together, routes requests)
- **SQLite** = the foundation (stores data permanently)
- **React** = the facade and interior (what people see and interact with)
- **Tailwind** = the finishing materials (colors, spacing, typography)

---

## 3. 📁 Project Structure

```
portfolio-tracker/
├── CLAUDE.md                    ← Context file for AI sessions
├── README.md                    ← Human-readable setup guide
│
├── backend/
│   ├── main.py                  ← FastAPI app entry point
│   ├── database.py              ← SQLAlchemy setup
│   ├── models.py                ← Database table definitions
│   ├── schemas.py               ← Data validation (Pydantic)
│   ├── pyproject.toml           ← Python dependencies (managed by uv)
│   ├── uv.lock                  ← Exact locked versions (commit this file)
│   │
│   ├── routers/                 ← API route groups
│   │   ├── portfolios.py        ← /api/portfolios
│   │   ├── holdings.py          ← /api/holdings
│   │   ├── transactions.py      ← /api/transactions
│   │   ├── dividends.py         ← /api/dividends
│   │   └── prices.py            ← /api/prices (market data)
│   │
│   └── services/
│       ├── calculations.py      ← Portfolio math (gain, yield, etc.)
│       └── price_fetcher.py     ← Yahoo Finance / API integration
│
├── frontend/
│   ├── package.json
│   ├── vite.config.js
│   ├── index.html
│   │
│   └── src/
│       ├── App.jsx              ← Root component, routing
│       ├── main.jsx             ← React entry point
│       │
│       ├── components/          ← Reusable UI pieces
│       │   ├── KPICard.jsx      ← "Valor actual: €3,780"
│       │   ├── StockTable.jsx   ← Holdings table
│       │   ├── TransactionForm.jsx
│       │   ├── DividendChart.jsx
│       │   └── BuyZoneTable.jsx
│       │
│       ├── pages/               ← Full page views
│       │   ├── Dashboard.jsx    ← Main control panel
│       │   ├── Transactions.jsx ← Transaction history
│       │   ├── Allocation.jsx   ← Buy zone analysis
│       │   └── Dividends.jsx    ← Dividend analysis
│       │
│       └── api/
│           └── client.js        ← Axios API calls to backend
│
└── portfolio.db                 ← SQLite database (auto-created)
```

### CLAUDE.md (copy this for every new session)

```markdown
# CLAUDE.md — Portfolio Tracker Context

## Project Summary
A personal investment portfolio tracker replacing an Excel file.
Two portfolios: "Main" (personal brokerage, ~€3,780) and "PEE" (Vinci employer plan, ~€6,499).

## Tech Stack
- Backend: Python 3.11 + FastAPI + SQLAlchemy + SQLite
- Package manager: uv (pyproject.toml + uv.lock — no pip, no venv, no requirements.txt)
- Frontend: React 18 + Vite + TailwindCSS + Recharts
- Database: SQLite (file: portfolio.db)

## Key Domain Concepts
- **Carteira** = Portfolio
- **Ações** = Stocks/shares
- **Dividendos** = Dividends
- **Compra/Venda** = Buy/Sell
- **Depósito** = Cash deposit to broker
- **Zona de compra** = Buy zone (price ≤ annual_dividend / required_yield)
- **PEE** = Plan d'Épargne Entreprise (Vinci employer savings plan)
- **Required yield** = 6.5% for FR/PT stocks (target minimum dividend return)

## Database Tables (current)
- portfolios: id, name (Main / PEE), created_at
- holdings: id, portfolio_id, ticker, company, quantity, avg_price, currency
- transactions: id, portfolio_id, date, ticker, company, action, quantity, price, total, notes
- dividends: id, portfolio_id, ticker, company, date, amount_per_share, total
- price_overrides: id, ticker, price, updated_at (for PEE funds with no market API)

## Current Holdings (Main Portfolio)
EPA:AMUN, EPA:RUI, EPA:TFI, EPA:COFA, ELI:NOS, EPA:SCR,
EPA:PSP5, ISIN:LU1832174962, EPA:ACA, EPA:RI, EPA:MMT

## API Endpoints (when built)
GET  /api/portfolios
GET  /api/portfolios/{id}/summary
GET  /api/holdings/{portfolio_id}
POST /api/transactions
GET  /api/transactions/{portfolio_id}
GET  /api/dividends/{portfolio_id}
GET  /api/allocation/{portfolio_id}

## Current Status
[UPDATE THIS EACH SESSION — e.g., "Module 3 complete. Working on Module 4 (Allocation page)."]
```

---

## 4. 🧩 Feature Breakdown: MVP + Future

> Think of each module like a floor of your building. You build them in order — you can't add the second floor before the first is solid.

### Phase 1 — MVP (Minimum Viable Product)

These are the features that make the app **genuinely useful from day one**.

| # | Module | What it does | Effort |
|---|---|---|---|
| M1 | **Project Setup** | Create folder structure, install dependencies, "Hello World" backend + frontend talking to each other | 🟢 Easy |
| M2 | **Database Schema** | Define all tables in SQLAlchemy, create the `.db` file, import your historical transactions from Excel | 🟡 Medium |
| M3 | **Transaction API** | POST/GET transactions (buy, sell, deposit, dividend). The core data entry. | 🟡 Medium |
| M4 | **Holdings Calculator** | Backend computes current holdings from transaction history (like Excel's Ações sheet) | 🟡 Medium |
| M5 | **Dashboard Page** | React page showing KPI cards: total value, invested, gain, dividends. Uses data from M4. | 🟢 Easy |
| M6 | **Stock Table** | Holdings table in React: ticker, company, quantity, avg price, current value, weight %, gain % | 🟡 Medium |
| M7 | **Transaction History Page** | Sortable, filterable list of all trades and deposits | 🟢 Easy |
| M8 | **Price Integration** | Fetch live prices via Yahoo Finance (yfinance Python library). Auto-refresh every 15 min. | 🔴 Hard |

### Phase 2 — Growth Features

| # | Module | What it does |
|---|---|---|
| M9 | **Dividend Analysis Page** | Table + bar chart of dividends received per company per year (like your "Analise dos dividendos" sheet) |
| M10 | **Buy Zone / Allocation Page** | Shows required yield, current yield, buy-zone price, distance from zone (replicates "Alocação" sheet) |
| M11 | **PEE Portfolio** | Separate view for Vinci PEE funds with manual price entry (no market API) |
| M12 | **Portfolio Growth Chart** | Line chart of total portfolio value over time |
| M13 | **Transaction Import** | Upload your Excel file to seed the database (one-time migration tool) |

### Phase 3 — Polish & Deploy

| # | Module | What it does |
|---|---|---|
| M14 | **Mobile Responsive Layout** | Looks great on phone |
| M15 | **Simple Auth** | Password-protect the app before making it public |
| M16 | **Deploy to Cloud** | Put it on Railway or Render so you can access it from anywhere |
| M17 | **Email Dividend Alerts** | Get notified when a dividend is expected |

---

## 5. 🔄 Development Workflow

### For every module, follow this ritual:

```
1. READ  → Understand what the module should do (I explain it)
2. PLAN  → We agree on the API contract or component interface
3. BUILD → I write the code, you paste it, you run it
4. TEST  → You test it manually in the browser or with /docs
5. COMMIT → You save the code (git commit)
6. UPDATE → You update the "Current Status" in CLAUDE.md
```

### How to maintain context between sessions

Claude has no memory between conversations. You fix this by:

1. **Always paste CLAUDE.md at the start of every session** (or attach the file)
2. **Update the "Current Status" line** after each session
3. **Paste relevant code snippets** if you're debugging something specific

Example session opener:
> "Here is my CLAUDE.md: [paste]. I finished M3 last session. Today I want to build M4 — the holdings calculator. Here is my current `models.py`: [paste]."

### How to ask me for each task

Use this template for best results:

```
Module: [M4 — Holdings Calculator]
Goal: [Calculate current holdings from transaction history]
Files involved: [models.py, routers/holdings.py, calculations.py]
Current code: [paste relevant files]
Problem/Question: [e.g., "How do I handle the case where a stock is fully sold?"]
```

---

## 6. 🚀 Next Immediate Steps

Here's exactly what to do after reading this plan:

### Step 1 — Install the tools (30 min)

```bash
# Install uv (one-time, global — replaces pip + venv forever)
# Mac / Linux:
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell):
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# Restart your terminal, then verify:
uv --version

# Check Node.js is installed (for the frontend)
node --version   # Should be 18+
# If not: https://nodejs.org/
```

> **What just happened?** uv is now installed globally. It will manage Python itself, your virtual environment, and all packages — you will never run `pip install`, `python -m venv`, or `source venv/bin/activate` again.

### Step 2 — Create the project skeleton (15 min)

```bash
# Create root folder
mkdir portfolio-tracker
cd portfolio-tracker

# --- BACKEND ---
# uv init creates pyproject.toml and installs a managed Python automatically
uv init backend
cd backend

# Add all backend dependencies in one command
# uv creates the virtualenv silently, installs everything, and writes uv.lock
uv add fastapi uvicorn sqlalchemy python-dotenv pydantic yfinance

# Run the app (uv handles the environment automatically — no activation needed)
uv run uvicorn main:app --reload

# --- FRONTEND (from the root folder) ---
cd ..
npm create vite@latest frontend -- --template react
cd frontend
npm install
npm install axios recharts tailwindcss @tailwindcss/vite
```

**The only files uv creates that you care about:**

| File | What it is | Commit to git? |
|---|---|---|
| `pyproject.toml` | Your dependency list — like `package.json` for Python | ✅ Yes |
| `uv.lock` | Exact pinned versions of every package | ✅ Yes |
| `.venv/` | The virtual environment folder (auto-created) | ❌ No (add to `.gitignore`) |

**Adding a new package later** — always use `uv add`, never `pip install`:
```bash
uv add httpx          # adds to pyproject.toml + uv.lock automatically
uv add --dev pytest   # dev-only dependency (not installed in production)
uv remove httpx       # removes cleanly
```

**When you clone the project on a new machine:**
```bash
cd backend
uv sync   # reads uv.lock and installs everything — one command, done
```

### Step 3 — Tell me you're ready

Once you have the folder structure, come back and say:

> "Setup complete. Here is my CLAUDE.md. Ready to build Module M2 — Database Schema."

I'll then write you the complete `database.py`, `models.py`, and the script to import your Excel data into SQLite.

---

## 💡 A Word of Encouragement

You're not starting from zero — you're starting from **domain expertise**. You already understand portfolios, dividend yields, buy zones, and transaction tracking better than most developers who would build this. That's your unfair advantage.

The code is just the translation layer. Every concept you already know in Excel, we'll translate into Python and React — one small, testable piece at a time.

The first three modules will feel slow. By Module 5, it'll feel natural. By Module 8, you'll be suggesting improvements I haven't thought of.

**Let's build it.** 🏗️
