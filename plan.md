# Protogen 200 Capstone — Business Analytics Dashboard

## Overview

A single-page analytics dashboard inspired by Shopify Admin and Google Analytics. Displays monthly business metrics for a fictional e-commerce store, with interactive filtering by month. Built in Viewify.

---

## Dataset

Fake data for January–December 2025. Each month includes four metrics: **revenue** (USD), **visitors**, **conversions** (%), and **orders**.

```json
[
  { "month": "Jan", "monthIndex": 1,  "revenue": 42300, "visitors": 8100, "conversions": 3.2, "orders": 259 },
  { "month": "Feb", "monthIndex": 2,  "revenue": 38750, "visitors": 7400, "conversions": 2.9, "orders": 215 },
  { "month": "Mar", "monthIndex": 3,  "revenue": 51200, "visitors": 9800, "conversions": 3.6, "orders": 353 },
  { "month": "Apr", "monthIndex": 4,  "revenue": 47600, "visitors": 9200, "conversions": 3.4, "orders": 313 },
  { "month": "May", "monthIndex": 5,  "revenue": 55900, "visitors": 11000, "conversions": 3.8, "orders": 418 },
  { "month": "Jun", "monthIndex": 6,  "revenue": 61400, "visitors": 12300, "conversions": 4.1, "orders": 504 },
  { "month": "Jul", "monthIndex": 7,  "revenue": 58800, "visitors": 11700, "conversions": 3.9, "orders": 456 },
  { "month": "Aug", "monthIndex": 8,  "revenue": 64200, "visitors": 13100, "conversions": 4.3, "orders": 563 },
  { "month": "Sep", "monthIndex": 9,  "revenue": 59700, "visitors": 12400, "conversions": 4.0, "orders": 496 },
  { "month": "Oct", "monthIndex": 10, "revenue": 72100, "visitors": 14800, "conversions": 4.5, "orders": 666 },
  { "month": "Nov", "monthIndex": 11, "revenue": 98400, "visitors": 21500, "conversions": 5.2, "orders": 1118 },
  { "month": "Dec", "monthIndex": 12, "revenue": 115600, "visitors": 24200, "conversions": 5.8, "orders": 1403 }
]
```

**Data notes:**
- Revenue spikes in Q4 reflect seasonal (holiday) shopping patterns.
- November and December are intentionally 40–80% above average to simulate Black Friday / Cyber Monday and Christmas.
- Conversions represent the percentage of visitors who completed a purchase.

---

## Layout (Vueify)

The dashboard uses a top-down, single-page layout with three vertical zones: a sticky filter bar, a KPI scorecard row, and a charts row. The filter bar is the global control — everything below it responds to the selected state.

---

### Zone 1 — Header + Filter Bar (sticky)

The header and month picker live together in a sticky top bar that stays visible as the user scrolls down to the charts.

**Header (left-aligned):**
- App title: **"Business Analytics — 2025"**
- Subtitle: *"Your store at a glance"* in muted text below

**Month Picker (right-aligned, same row as header):**
- Component: segmented button group
- Options: `All` · `Jan` · `Feb` · `Mar` · `Apr` · `May` · `Jun` · `Jul` · `Aug` · `Sep` · `Oct` · `Nov` · `Dec`
- Default: **`All`** selected on load
- The active selection is visually filled with the accent color; inactive buttons are outlined/ghost style
- Behavior:
  - `All` selected → all 12 months of data flow into every card and chart (12-month view)
  - Single month selected → all cards and both charts filter to that month's data only; cards also show a trend badge vs. the prior month

---

## Zone 2 — KPI Scorecard Cards

Four cards in a **4-column grid** (2-column on tablet, 1-column on mobile), displayed below the filter bar with consistent spacing.

Each card is a clean scorecard — the big number is the hero, everything else is secondary.

| Card | Metric | Default (All) | Single month |
|------|--------|---------------|--------------|
| Revenue | Total revenue | Annual sum | That month's revenue |
| Visitors | Total visitors | Annual sum | That month's visitors |
| Conversions | Avg conversion rate | Annual average | That month's rate |
| Orders | Total orders | Annual sum | That month's orders |

**Card anatomy:**
- Small colored icon top-left (dollar sign / person / arrow-up / shopping cart), one per card with a distinct accent color
- Metric label in small uppercase muted text (e.g., "TOTAL REVENUE")
- **Large display number** as the primary element — formatted (`$XX,XXX` / `XX,XXX` / `X.X%`)
- Trend badge — only visible when a single month is selected; shows `▲ +X%` or `▼ -X%` vs. the prior month in green/red; hidden in the All view to keep the scorecard clean

---

## Zone 3 — Charts Row

Two charts side by side in a **2-column grid** (stacked full-width on mobile), below the KPI cards.

### Left — Bar Chart: Monthly Revenue

- Title: **"Monthly Revenue"**
- Default (All): 12 bars, one per month, Jan–Dec on the X-axis
- Single month selected: the chart keeps all 12 bars and the same X-axis structure — the selected month's bar renders at full color, all other bars drop to ~25% opacity
- X-axis: month labels (Jan–Dec)
- Y-axis: revenue in USD, formatted as `$XXK`
- Tooltip on hover: `"[Month]: $XX,XXX"`
- Bar color: teal/green accent

### Right — Line Chart: Visitors Over Time

- Title: **"Visitor Trends"**
- Default (All): smooth curve across all 12 months with a filled area beneath the line
- Single month selected: the chart keeps all 12 data points and the full line — the selected month's point enlarges and renders a label; the rest of the line and area dim to ~25% opacity
- X-axis: month labels (Jan–Dec)
- Y-axis: visitor count, formatted as `XXK`
- Tooltip on hover: `"[Month]: X,XXX visitors"`
- Line and fill color: indigo/blue

---

## Zone 4 — Footer

- Single line of muted small text, centered: *"Data is simulated for demonstration purposes. Protogen 200 Capstone — 2025."*

---

## Interactions

- Month picker in the app bar filters EVERYTHING — summary cards show that month's numbers, charts filter to that month's data
- When "All" is selected, summary cards show yearly totals/averages and charts show all 12 months
- Cards show a small up/down arrow and color indicating change from the previous month (only when a single month is selected)

---

## Style

- Light mode — clean, modern design using Vuetify's light theme
- Minimal and spacious — lots of whitespace, no visual clutter
- Accessible color palette: all text/background combinations meet WCAG AA contrast ratios (4.5:1 minimum); trend indicators use both color and icon (▲/▼) so color is never the only signal
- Charts use a cohesive, limited color palette — not rainbow; two or three colors max across the whole dashboard
- Mobile responsive — cards stack on small screens

---

## Tech

- Vue 3 + TypeScript + Vuetify 3
- Chart.js via vue-chartjs for all charts
- Fake data from a local JSON file (no API calls)
- Single page — no routing needed for this app
