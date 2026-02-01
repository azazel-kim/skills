---
name: naver-stock
description: Fetch text-based real-time stock prices (KRX, Overseas) using Naver Finance.
metadata: {"clawdbot":{"emoji":"📈","os":["mac","linux","windows"],"requires":{"bins":["node"]}}}
---

# Naver Stock

Fetch real-time stock prices for domestic (KRX) and overseas markets using Naver Finance.

## Usage

Run the bundled script with a stock name or code.

```bash
node index.cjs "삼성전자"
node index.cjs "AAPL"
```

## Output Format

Returns a JSON object with price details.

```json
{
  "name": "삼성전자",
  "code": "005930",
  "price": 160500,
  "change": -200,
  "changePercent": -0.12,
  "nxtPrice": 160800,
  "nxtChange": 100,
  "nxtChangePercent": 0.06,
  "currency": "KRW"
}
```

## Examples

### Domestic Stock
```bash
node index.cjs 005930
```

### Overseas Stock
```bash
node index.cjs "Tesla"
```

### Exchange Rate
```bash
node index.cjs "USD"
node index.cjs "엔"
```
