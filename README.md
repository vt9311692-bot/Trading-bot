# Binance Futures Trading Bot

A clean, well-structured Python CLI trading bot for the Binance Futures (USDT-M) testnet.

> **No Binance account required** — the bot ships with a built-in **Mock Mode** that
> simulates the Binance Futures Testnet API locally with realistic price jitter and
> fill logic. Switch to Live Testnet mode any time by passing `--live` with your API
> credentials.

---

## Features

| Feature | Details |
|---|---|
| Order types | **MARKET**, **LIMIT**, **STOP_MARKET** (bonus) |
| Sides | **BUY** and **SELL** |
| Mock mode | Full local simulation — no API keys needed |
| Live mode | Real Binance Futures Testnet (`https://testnet.binancefuture.com`) |
| Validation | Symbol format, side, order type, quantity, price, stop-price |
| Logging | Rotating file log (`logs/trading_bot.log`) + console output |
| Error handling | Input errors, API errors, network failures |

---

## Project Structure

```
trading_bot/
├── bot/
│   ├── __init__.py
│   ├── client.py          # Binance client wrapper (mock + live)
│   ├── orders.py          # Order placement logic & formatting
│   ├── validators.py      # Input validation
│   └── logging_config.py  # Rotating file + console logging
├── logs/
│   └── trading_bot.log    # Auto-created on first run
├── cli.py                 # CLI entry point (argparse)
├── requirements.txt
└── README.md
```

---

## Setup

### Requirements

- Python **3.10+** (uses `float | None` union type hints)
- No third-party packages needed for **mock mode**
- `requests` needed only for **live testnet mode**

```bash
# Clone / unzip the project, then:
cd trading_bot

# (Optional) create a virtual environment
python -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate

# Install dependencies (only needed for --live mode)
pip install -r requirements.txt
```

For **live testnet** mode, uncomment `requests` in `requirements.txt` first:
```bash
# Edit requirements.txt and uncomment:
#   requests>=2.31.0
pip install -r requirements.txt
```

---

## How to Run

### Mock Mode (default — no API keys needed)

**MARKET BUY:**
```bash
python cli.py --symbol BTCUSDT --side BUY --type MARKET --quantity 0.01
```

**LIMIT SELL:**
```bash
python cli.py --symbol ETHUSDT --side SELL --type LIMIT --quantity 0.5 --price 3400
```

**STOP_MARKET BUY (bonus order type):**
```bash
python cli.py --symbol SOLUSDT --side BUY --type STOP_MARKET --quantity 1 --stop-price 160
```

### Live Testnet Mode

```bash
python cli.py --symbol BTCUSDT --side BUY --type MARKET --quantity 0.001 \
              --api-key YOUR_API_KEY --api-secret YOUR_API_SECRET --live
```

### All CLI Arguments

| Argument | Required | Description |
|---|---|---|
| `--symbol` | ✅ | Trading pair, e.g. `BTCUSDT` |
| `--side` | ✅ | `BUY` or `SELL` |
| `--type` | ✅ | `MARKET`, `LIMIT`, or `STOP_MARKET` |
| `--quantity` | ✅ | Order quantity, e.g. `0.01` |
| `--price` | LIMIT only | Limit price |
| `--stop-price` | STOP_MARKET only | Stop trigger price |
| `--api-key` | live only | Binance API key |
| `--api-secret` | live only | Binance API secret |
| `--live` | — | Use real testnet (default: mock) |
| `--log-dir` | — | Log directory (default: `logs/`) |

---

## Sample Output

```
┌─ Order Request ─────────────────────────────
│  Symbol     : BTCUSDT
│  Side       : BUY
│  Type       : MARKET
│  Quantity   : 0.01
└─────────────────────────────────────────────
  Current mark price for BTCUSDT: 67432.50

┌─ Order Response ────────────────────────────  ⚙  [MOCK — simulated response]
│  Order ID   : 1741234567890
│  Symbol     : BTCUSDT
│  Side       : BUY
│  Type       : MARKET
│  Status     : FILLED
│  Orig Qty   : 0.01
│  Exec Qty   : 0.01
│  Avg Price  : 67419.33
└─────────────────────────────────────────────

✅  Order submitted successfully! Order ID: 1741234567890
```

---

## Assumptions & Notes

1. **Mock mode** is the default and requires zero configuration. It uses hardcoded
   mid-prices with ±0.05 % random jitter and simulates fill logic (MARKET orders
   always fill; LIMIT orders fill when the limit price is on the favourable side of
   the mark price; STOP_MARKET orders are placed as `NEW`).

2. **Supported symbols in mock mode:** `BTCUSDT`, `ETHUSDT`, `BNBUSDT`, `SOLUSDT`,
   `XRPUSDT`, `DOGEUSDT`. Any `<BASE>USDT` symbol is accepted by the validator; only
   the above six have simulated prices.

3. In **live mode** the bot targets `https://testnet.binancefuture.com`. You need a
   Binance Futures Testnet account and API credentials. The bot is **not** intended
   for real (mainnet) trading.

4. Log files rotate at 5 MB and keep 3 backups. Logs are written to `logs/trading_bot.log`
   by default.
