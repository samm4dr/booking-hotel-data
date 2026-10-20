# Scrape Booking.com Hotel Data: Prices, Reviews and Competitor Rates

Hotel rate parity, competitor pricing, review mining — Booking.com data feeds revenue management and travel intelligence. Direct scraping hits CAPTCHAs fast; Apify actors handle it. Here's the pipeline.

> 🤖 **Try it live** — chat with a Poe bot that generates the exact pipeline for your market: [Lead List Generator](https://poe.com/leadlistgenerator) · [Job Postings Miner](https://poe.com/jobpostingsminer) · [Amazon Product Miner](https://poe.com/amazonproductminer) · [SERP Rank Tracker](https://poe.com/serpranktracker) · [Social Lead Finder](https://poe.com/socialleadfinder)

## The stack

- **[Apify](https://console.apify.com/sign-up?fpr=dx06p)** — Booking.com actors for hotels, prices, reviews. Free monthly credits.
- **Python** — daily rate snapshots, review mining.

## Step 1 — Token

[Create an Apify account](https://console.apify.com/sign-up?fpr=dx06p) → Settings → Integrations → `export APIFY_TOKEN=...`

## Step 2 — Pull hotels in a destination

```bash
curl -X POST "https://api.apify.com/v2/acts/santamaria-automations~booking-com-scraper/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "destination": "Miami",
    "checkIn": "2026-11-15",
    "checkOut": "2026-11-17",
    "adults": 2,
    "maxItems": 200
  }'
```

Fields: `name`, `address`, `price`, `currency`, `reviewScore`, `reviewCount`, `stars`, `url`, `imageUrls`.

## Step 3 — Price-gap analysis

```python
from apify_client import ApifyClient

client = ApifyClient("APIFY_TOKEN")
run = client.actor("santamaria-automations/booking-com-scraper").call(run_input={
    "destination": "Miami Beach",
    "checkIn": "2026-11-15", "checkOut": "2026-11-17",
    "adults": 2, "maxItems": 300,
})

hotels = []
for h in client.dataset(run["defaultDatasetId"]).iterate_items():
    price, score = h.get("price"), h.get("reviewScore") or 0
    if price and score:
        hotels.append((score, price, h["name"]))

# high rating + low price = value leaders to monitor/beat
for s, p, n in sorted(hotels, key=lambda x: -x[0]/x[1]):
    print(f"{s:>4.1f}★ ${p:>5.0f} {n[:55]}")
```

## The plays

- **Hoteliers** — daily competitor rates → revenue management without a $300/month tool.
- **Travel affiliates** — price-drop feeds for deal newsletters and booking pages.
- **Review intel** — `zhorex/booking-reviews-scraper` on your property AND competitors → what guests praise/complain about = your amenity + marketing roadmap.

## Useful actors

| Actor | Job | Users |
|---|---|---|
| `santamaria-automations/booking-com-scraper` | Hotel search + rates | 1.2k |
| `rl1987/booking-api-scraper` | Prices, room rates, availability | 336 |
| `zhorex/booking-reviews-scraper` | Full review text + scores | 49 |

## More resources

- [apify-skills](https://github.com/g4dr/apify-skills) — run this from a prompt: `npx skills add g4dr/apify-skills`
- [awesome-web-scraping](https://github.com/g4dr/awesome-web-scraping)
- Free credits: [Apify signup](https://console.apify.com/sign-up?fpr=dx06p)
