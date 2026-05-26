# 📐 Financial Equations — Portfolio Tracker

> Personal reference: every formula found in the Excel spreadsheet, explained from first principles.
> Written for someone with an engineering background who wants to understand the *why*, not just the *what*.

---

## How to read this document

Each equation is presented in four parts:

1. **What it measures** — the concept in plain English
2. **The formula** — in mathematical notation
3. **Excel → Python translation** — how it becomes code
4. **A concrete example** — using real numbers from your spreadsheet

---

## 1. Average Purchase Price (Preço Médio)

### What it measures
When you buy shares of the same company multiple times at different prices, this tells you the true average price you paid per share across all purchases. Think of it like averaging the unit price of materials bought in multiple batches on a construction project.

### Formula

$$\bar{P} = \frac{\sum_{i=1}^{n} (Q_i \times P_i)}{\sum_{i=1}^{n} Q_i}$$

Where:
- $Q_i$ = quantity bought in transaction $i$
- $P_i$ = price paid per share in transaction $i$
- $n$ = total number of buy transactions for this stock

### Excel → Python

```python
# From the transaction history, for one stock (e.g. EPA:COFA / Coface):
# Transaction 1: 7 shares @ €14.88
# Transaction 2: 6 shares @ €14.838
# Transaction 3: 6 shares @ €15.371

def average_price(transactions: list[dict]) -> float:
    buys = [t for t in transactions if t["action"] == "Compra"]
    total_cost = sum(t["quantity"] * t["price"] for t in buys)
    total_quantity = sum(t["quantity"] for t in buys)
    return total_cost / total_quantity if total_quantity > 0 else 0
```

### Concrete example — Coface (EPA:COFA)

| Transaction | Quantity | Price | Cost |
|---|---|---|---|
| Buy 1 | 7 | €14.88 | €104.16 |
| Buy 2 | 6 | €14.838 | €89.03 |
| Buy 3 | 6 | €15.371 | €92.23 |
| **Total** | **19** | | **€285.42** |

$$\bar{P} = \frac{285.42}{19} = \textbf{€15.02 per share}$$

✅ Matches your Excel: `Preço médio = 15.02210526`

> **Why it matters:** Your average price is your personal break-even point. If the stock trades above this, you're in profit. Below, you're at a loss — even if the stock is objectively "up" since you started investing.

---

## 2. Invested Value (Valor Comprado)

### What it measures
The total amount of real money you put into a stock. This is your cost basis — what you actually spent.

### Formula

$$V_{invested} = Q_{total} \times \bar{P}$$

Where:
- $Q_{total}$ = total shares currently held
- $\bar{P}$ = average purchase price

### Excel → Python

```python
def invested_value(quantity: float, avg_price: float) -> float:
    return quantity * avg_price
```

### Concrete example — Coface

$$V_{invested} = 19 \times 15.02 = \textbf{€285.42}$$

---

## 3. Current Value (Valor Atual)

### What it measures
What your position is worth *right now* at market prices. This changes every second the market is open.

### Formula

$$V_{current} = Q_{total} \times P_{current}$$

### Excel → Python

```python
def current_value(quantity: float, current_price: float) -> float:
    return quantity * current_price
```

### Concrete example — Coface

$$V_{current} = 19 \times 16.45 = \textbf{€312.55}$$

---

## 4. Unrealized Gain/Loss (Valorização €)

### What it measures
The profit or loss on a position you **still hold**. It's called "unrealized" because it's not real money yet — you only realize it when you sell. Like owning land that has appreciated in value: you're richer on paper, but you only get the cash when you sell the land.

### Formula

$$G_{unrealized} = V_{current} - V_{invested}$$

### Excel → Python

```python
def unrealized_gain(current_value: float, invested_value: float) -> float:
    return current_value - invested_value
```

### Concrete example — Coface

$$G_{unrealized} = 312.55 - 285.42 = \textbf{+€27.13}$$

Negative example — TFI (EPA:TFI):
$$G_{unrealized} = 102.00 - 107.53 = \textbf{-€5.53}$$

---

## 5. Unrealized Gain Percentage (Valorização %)

### What it measures
The same gain expressed as a percentage of your original investment. This lets you compare performance across positions of different sizes — a €27 gain on a €285 investment is much more impressive than a €27 gain on a €2,850 investment.

### Formula

$$G_{\%} = \frac{V_{current} - V_{invested}}{V_{invested}} \times 100$$

### Excel → Python

```python
def unrealized_gain_percent(current_value: float, invested_value: float) -> float:
    if invested_value == 0:
        return 0
    return (current_value - invested_value) / invested_value * 100
```

### Concrete example — Coface

$$G_{\%} = \frac{312.55 - 285.42}{285.42} \times 100 = \frac{27.13}{285.42} \times 100 = \textbf{+9.50\%}$$

✅ Matches your Excel: `Valorização % = 0.09505290449` (= 9.50%)

---

## 6. Portfolio Weight (Peso da Ação)

### What it measures
What percentage of your total portfolio is concentrated in one stock. A civil engineering analogy: it's the load distribution across columns — if one column carries 50% of the load, a failure there brings down the whole structure. Diversification means spreading the weight.

### Formula

$$W_i = \frac{V_{current,i}}{\sum_{j=1}^{n} V_{current,j}}$$

Where $i$ is one specific stock and the denominator is the sum of all current values across all $n$ stocks.

### Excel → Python

```python
def portfolio_weights(holdings: list[dict]) -> dict:
    total = sum(h["current_value"] for h in holdings)
    return {
        h["ticker"]: h["current_value"] / total
        for h in holdings
        if total > 0
    }
```

### Concrete example

Total portfolio value = €4,374.03 (sum of all positions)

| Stock | Current Value | Weight |
|---|---|---|
| S&P 500 Amundi | €966.28 | 22.09% |
| Small caps EUR | €945.84 | 21.62% |
| Credit Agricole | €596.36 | 13.63% |
| Pernod Ricard | €649.20 | 14.84% |
| ... | ... | ... |

> **Why it matters:** Your Excel shows S&P 500 + Small caps alone represent ~44% of the portfolio. That's a meaningful concentration in index funds rather than individual dividend stocks.

---

## 7. Dividend Yield (Rendimento dos Dividendos)

### What it measures
The annual income you receive from a stock expressed as a percentage of its current price. If a stock pays €2 per share and costs €40, you get 5% of your investment back in cash every year — regardless of whether the stock price goes up or down. This is the core metric for a dividend investment strategy.

### Formula

$$Y_{div} = \frac{D_{annual}}{P_{current}}$$

Where:
- $D_{annual}$ = total dividends paid per share in the last 12 months (or next 12 months if forecasting)
- $P_{current}$ = current market price per share

### Excel → Python

```python
def dividend_yield(annual_dividend_per_share: float, current_price: float) -> float:
    if current_price == 0:
        return 0
    return annual_dividend_per_share / current_price
```

### Concrete example — Coface (EPA:COFA)

$$Y_{div} = \frac{2.15}{16.45} = 0.1307 = \textbf{13.07\%}$$

✅ Matches your Excel: `Rentabilidade atual estimada = 0.1306990881`

For comparison, NOS (ELI:NOS):
$$Y_{div} = \frac{1.40}{5.26} = 0.2662 = \textbf{26.62\%}$$

> **Caution:** A very high dividend yield (like NOS at 26%) can be a warning sign. It sometimes means the market expects the dividend to be cut, or the company is in trouble. The yield is high because the price has fallen, not because dividends increased. Always check the payout ratio and earnings.

---

## 8. Required Yield (Rentabilidade Exigida)

### What it measures
The **minimum** dividend yield you accept before buying a stock. This is your personal filter — you set it as a strategy rule. Your spreadsheet uses **6.5% for all stocks** (both French and Portuguese, though the column suggests country-specific rates were considered).

This is not a calculation — it's a **policy decision**. You decided: "I will not buy a stock unless it pays me at least 6.5% per year in dividends at my purchase price."

### Why 6.5%?

This rate represents your opportunity cost. If a risk-free government bond pays ~3-4%, you need a meaningful premium to justify the risk of owning individual company stocks. 6.5% is a common threshold in dividend investing communities to target.

```python
# This is a configuration constant, not a formula
REQUIRED_YIELD_BY_COUNTRY = {
    "FR": 0.065,  # France
    "PT": 0.065,  # Portugal
    # Future: could differentiate by country or sector
}
```

---

## 9. Buy Zone Price (Zona de Compra)

### What it measures
The **maximum price you should pay** for a stock to guarantee your required yield. If you want at least 6.5% annual dividend income, and a company pays €2.15 per share per year, you must buy at or below a certain price. Above that price, the yield drops below your minimum threshold.

This is the core valuation tool in your spreadsheet. Think of it like a "maximum bid" in a public works tender — you calculate the maximum you can spend and still make the project financially viable.

### Formula

$$P_{buy zone} = \frac{D_{annual}}{Y_{required}}$$

This is derived directly from the dividend yield formula, solving for price:

$$Y = \frac{D}{P} \implies P = \frac{D}{Y}$$

### Excel → Python

```python
def buy_zone_price(annual_dividend_per_share: float, required_yield: float) -> float:
    if required_yield == 0:
        return 0
    return annual_dividend_per_share / required_yield
```

### Concrete example — Coface

$$P_{buy zone} = \frac{2.15}{0.065} = \textbf{€33.08}$$

✅ Matches your Excel: `ZONA DE COMPRA = 33.07692308`

Current price is €16.45 → well below €33.08 → **IN the buy zone** ✅

For Amundi (EPA:AMUN):
$$P_{buy zone} = \frac{4.25}{0.065} = \textbf{€65.38}$$

Current price is €87.25 → above €65.38 → **NOT in the buy zone** ❌

> **Intuition:** This is like calculating how much you can pay for a rental property to guarantee a minimum rental yield. If you need 6.5% rental income, and the rent is €1,300/month (€15,600/year), you should pay no more than €15,600 ÷ 0.065 = €240,000 for the property. Same exact logic.

---

## 10. Distance from Buy Zone (Distância da Zona de Compra)

### What it measures
How far the current price is from the buy zone limit, expressed as a percentage. Positive means you're inside the zone (cheap enough). Negative means you're outside (too expensive).

### Formula

$$D_{buy zone} = \frac{P_{buy zone} - P_{current}}{P_{current}}$$

### Excel → Python

```python
def distance_from_buy_zone(buy_zone_price: float, current_price: float) -> float:
    if current_price == 0:
        return 0
    return (buy_zone_price - current_price) / current_price
```

### Concrete examples

**Coface** (IN zone): 
$$D = \frac{33.08 - 16.45}{16.45} = \frac{16.63}{16.45} = \textbf{+101.1\%}$$

✅ Matches Excel: `+1.010755202` (= +101.1%)

**Amundi** (NOT in zone):
$$D = \frac{65.38 - 87.25}{87.25} = \frac{-21.87}{87.25} = \textbf{-25.1\%}$$

✅ Matches Excel: `-0.2506061274` (= -25.1%)

> **Reading the signal:** A distance of +101% (Coface) means the stock would have to *double* before leaving the buy zone. A distance of -25% (Amundi) means the price needs to fall 25% before it becomes attractive. The bigger the positive number, the more undervalued the stock is relative to your yield target.

---

## 11. Weighted Portfolio Yield (Rendimento Ponderado)

### What it measures
The dividend yield of each stock, adjusted by how much of your portfolio it represents. This lets you compute the overall expected dividend income of your entire portfolio as a single percentage.

### Formula — Per Stock

$$Y_{weighted,i} = Y_{div,i} \times W_i$$

Where:
- $Y_{div,i}$ = dividend yield of stock $i$
- $W_i$ = portfolio weight of stock $i$ (from equation 6)

### Formula — Portfolio Total

$$Y_{portfolio} = \sum_{i=1}^{n} Y_{weighted,i} = \sum_{i=1}^{n} (Y_{div,i} \times W_i)$$

### Excel → Python

```python
def portfolio_yield(holdings: list[dict]) -> float:
    """
    Each holding needs:
    - dividend_yield: float (annual dividend / current price)
    - weight: float (position value / total portfolio value)
    """
    return sum(h["dividend_yield"] * h["weight"] for h in holdings)
```

### Concrete example — SCOR SE (EPA:SCR)

- Dividend yield: $2.17 / 32.14 = 6.75\%$
- Portfolio weight: $5.14\%$
- Weighted contribution: $0.0675 \times 0.0514 = \textbf{0.00347}$ (= 0.347%)

Your Excel shows the total portfolio yield is **4.30%** — meaning for every €100 invested across all stocks, you expect to receive ~€4.30 in dividends per year.

---

## 12. Weighted Average Yield (Equal-Weight Scenario)

### What it measures
Your spreadsheet computes two scenarios: your actual weighted yield (using real portfolio weights), and what the yield would be if you invested **equally** in all stocks. This lets you see whether your current allocation is optimal for dividend income.

### Formula — Equal Weight

$$Y_{equal} = \frac{1}{n} \sum_{i=1}^{n} Y_{div,i}$$

This is a simple average of all dividend yields, pretending each stock has the same weight.

### Excel → Python

```python
def equal_weight_yield(holdings: list[dict]) -> float:
    if not holdings:
        return 0
    return sum(h["dividend_yield"] for h in holdings) / len(holdings)
```

### Your Excel values

| Scenario | Yield |
|---|---|
| Actual portfolio yield (current weights) | **4.30%** |
| Equal-weight yield (all stocks same size) | **7.48%** |
| Equal-weight yield (buy-zone stocks only) | **8.50%** |

> **What this tells you:** Your actual yield (4.30%) is lower than the equal-weight yield (7.48%) because your largest positions (S&P 500 ETF at 22%, Small caps at 21%) pay very low dividends. If you rebalanced toward the high-yield dividend stocks, your income would increase significantly.

---

## 13. Daily Variation (Variação do Dia)

### What it measures
How much a stock moved today compared to yesterday's closing price. Your spreadsheet shows both the percentage and the absolute euro amount.

### Formula — Percentage

$$\Delta_{\%} = \frac{P_{today} - P_{yesterday}}{P_{yesterday}}$$

### Formula — Absolute (€)

$$\Delta_{\euro} = P_{today} - P_{yesterday}$$

Or equivalently:

$$\Delta_{\euro} = P_{today} \times \Delta_{\%}$$

### Excel → Python

```python
def daily_variation_percent(today: float, yesterday: float) -> float:
    if yesterday == 0:
        return 0
    return (today - yesterday) / yesterday

def daily_variation_euro(today: float, yesterday: float) -> float:
    return today - yesterday
```

> **Note:** These values come from a market data API (Yahoo Finance, etc.) — they are not calculated from your transaction history. The app fetches them live.

---

## 14. Total Dividends Received (Dividendos Totais)

### What it measures
The cumulative cash income received from dividends since the portfolio was created. This is simply a sum of all dividend transactions in the history.

### Formula

$$D_{total} = \sum_{i \in \text{Dividendos}} T_i$$

Where $T_i$ is the total amount of each dividend transaction.

### Excel → Python

```python
def total_dividends(transactions: list[dict]) -> float:
    return sum(
        t["total"]
        for t in transactions
        if t["action"] == "Dividendos"
    )
```

### Your numbers

| Stock | Dividends Received |
|---|---|
| TFI (EPA:TFI) | €9.45 |
| NOS (ELI:NOS) | €4.97 |
| SCOR SE (EPA:SCR) | €13.30 |
| Groupe M6 (EPA:MMT) | €12.50 |
| **Total** | **€40.22** |

---

## 15. Dividend Contribution (% Contribuição)

### What it measures
What percentage of your total dividend income came from each stock. Tells you which stocks are actually working for you as income generators.

### Formula

$$C_i = \frac{D_i}{D_{total}}$$

### Excel → Python

```python
def dividend_contribution(stock_dividends: float, total_dividends: float) -> float:
    if total_dividends == 0:
        return 0
    return stock_dividends / total_dividends
```

### Concrete example

| Stock | Dividends | Contribution |
|---|---|---|
| SCOR SE | €13.30 | 33.1% |
| Groupe M6 | €12.50 | 31.1% |
| TFI | €9.45 | 23.5% |
| NOS | €4.97 | 12.4% |
| **Total** | **€40.22** | **100%** |

---

## 16. Portfolio Cash Position (Cash)

### What it measures
The uninvested cash sitting in your brokerage account. In your system, cash is tracked through the transaction history itself: deposits add cash, purchases subtract cash, dividends and sales add cash.

### Formula

$$Cash = \sum_{\text{Depósito}} T_i + \sum_{\text{Venda}} T_i + \sum_{\text{Dividendos}} T_i - \sum_{\text{Compra}} T_i - \sum_{\text{Retirada}} T_i$$

### Excel → Python

```python
def cash_balance(transactions: list[dict]) -> float:
    cash_in = {"Depósito", "Venda", "Dividendos"}
    cash_out = {"Compra", "Retirada"}
    
    balance = 0
    for t in transactions:
        if t["action"] in cash_in:
            balance += t["total"]
        elif t["action"] in cash_out:
            balance -= t["total"]
    return balance
```

### Your numbers

Total deposited: 5 deposits × various amounts = €3,350 total in
Total spent on purchases: ~€3,944 out
Result: **-€593.65** (negative cash = money owed to broker, or margin)

> **Note:** A negative cash balance in your spreadsheet likely means your brokerage allows fractional investing with a small credit, or there's rounding across transactions.

---

## Summary Table — All Equations

| # | Equation | Formula | Service function |
|---|---|---|---|
| 1 | Average price | $\sum(Q_i \times P_i) / \sum Q_i$ | `average_price()` |
| 2 | Invested value | $Q \times \bar{P}$ | `invested_value()` |
| 3 | Current value | $Q \times P_{current}$ | `current_value()` |
| 4 | Unrealized gain (€) | $V_{current} - V_{invested}$ | `unrealized_gain()` |
| 5 | Unrealized gain (%) | $(V_{current} - V_{invested}) / V_{invested}$ | `unrealized_gain_percent()` |
| 6 | Portfolio weight | $V_i / \sum V_j$ | `portfolio_weights()` |
| 7 | Dividend yield | $D_{annual} / P_{current}$ | `dividend_yield()` |
| 8 | Required yield | Policy constant (6.5%) | `REQUIRED_YIELD` |
| 9 | Buy zone price | $D_{annual} / Y_{required}$ | `buy_zone_price()` |
| 10 | Distance from buy zone | $(P_{buy zone} - P_{current}) / P_{current}$ | `distance_from_buy_zone()` |
| 11 | Weighted portfolio yield | $\sum (Y_i \times W_i)$ | `portfolio_yield()` |
| 12 | Equal-weight yield | $\sum Y_i / n$ | `equal_weight_yield()` |
| 13 | Daily variation | $(P_{today} - P_{yesterday}) / P_{yesterday}$ | From market API |
| 14 | Total dividends | $\sum T_{\text{Dividendos}}$ | `total_dividends()` |
| 15 | Dividend contribution | $D_i / D_{total}$ | `dividend_contribution()` |
| 16 | Cash balance | $\sum T_{in} - \sum T_{out}$ | `cash_balance()` |
