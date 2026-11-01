# EV Market 2026: Power BI Dashboard

A three-page interactive Power BI report built on the cleaned EV market data from SQL Server. It covers overall market trends, brand and model performance, and the relationship between price, range and customer rating.

> The dataset appears to be simulated, so the insights below illustrate the analysis and are not real market facts.

**Previous stage:** [SQL cleaning and analysis](../sql/README.md)

---

## Dashboard preview

### Page 1: Market overview

![Market Overview Dashboard](screenshots/01_market_overview.png)

### Page 2: Brands and models

![Brands and Models Dashboard](screenshots/02_brands_and_models.png)

### Page 3: Price, range and value

![Price Range and Value Dashboard](screenshots/03_price_range_value.png)

---

## Data source

- Source: `dbo.vw_ev_market` in the `EV_Market` SQL Server database
- Connection mode: Import
- Model: one flat table named `EV Market` (2,000 rows), so no relationships were needed
- A separate `_Measures` table holds all DAX measures

## What each page shows

### Page 1: Market overview

- **Slicers:** model year, market segment, country of origin
- **KPI cards:** average rating, weighted average price, total revenue, total units
- **Combo chart:** total units (columns) and weighted average price (line) by model year
- **Bar chart:** total units by brand
- **Treemap:** total units by market segment
- **Bar chart:** total units by body type

Headline figures: 259M units, a weighted average price of about $78K and an average rating of 3.57.

### Page 2: Brands and models

- **Brand slicer** (button style)
- **Matrix:** total units by brand and model year with a colour scale
- **Table:** brand, listings, total units, market share % and average rating
- **Bar chart:** total units by model

### Page 3: Price, range and value

- **Slicers:** body type and drive type
- **Scatter chart:** average range against average price by brand and market segment, sized by total units
- **Bar chart:** average price per mile of range by brand
- **Bar chart:** average rating by range band
- **Column chart:** total units by price band

## Key observations

- **Market growth:** units rise each year from 2020 to a peak in 2025, then drop sharply in 2026. The weighted average price climbs through 2024 and eases slightly afterwards.
- **Brand concentration:** Tesla holds 39.1% of units, BYD 19.5% and Volkswagen 18.6%, so the top three account for roughly three quarters of the market.
- **Segments and body types:** Premium is the largest segment by units. Truck is the largest body type and Van the smallest.
- **Range and satisfaction:** average customer rating increases with range band, from roughly 3.4 for vehicles under 200 miles to just under 4.0 for 400+ miles.
- **Value:** Lucid has by far the highest price per mile of range, followed by Porsche and Mercedes. The Budget segment sits at the low-price, mid-range end of the scatter chart while Luxury sits at the top.
- **Price bands:** the $60K to $100K band sells the most units, and the under $30K band the fewest.

## DAX measures

```dax
Total Units = SUM('EV Market'[annual_sales_units])

Total Revenue = SUM('EV Market'[est_revenue_usd])

Listings = COUNTROWS('EV Market')

Avg Price = AVERAGE('EV Market'[price_usd])

Weighted Avg Price = DIVIDE([Total Revenue], [Total Units])

Avg Range (mi) = AVERAGE('EV Market'[range_miles])

Avg Rating = AVERAGE('EV Market'[customer_rating])

Avg Efficiency (mi/kWh) = AVERAGE('EV Market'[efficiency_mi_per_kwh])

Avg Price per Mile = AVERAGE('EV Market'[price_per_mile_range])

Market Share % =
DIVIDE(
    [Total Units],
    CALCULATE([Total Units], ALLSELECTED('EV Market'[brand]))
)

Prev Year Units =
CALCULATE(
    [Total Units],
    FILTER(
        ALL('EV Market'[model_year]),
        'EV Market'[model_year] = MAX('EV Market'[model_year]) - 1
    )
)

Units YoY % = DIVIDE([Total Units] - [Prev Year Units], [Prev Year Units])

Flagged Rows % =
DIVIDE(
    CALCULATE(COUNTROWS('EV Market'), 'EV Market'[dup_key_flag] = 1),
    COUNTROWS('EV Market')
)
```

Notes on the choices:

- **Weighted Avg Price** divides revenue by units, so a high-volume model counts for more than a niche one. A simple average treats them equally.
- **Market Share %** uses `ALLSELECTED` so the shares always add up to 100% across whatever brands the viewer has selected, instead of across all brands.

## Modelling and design choices

- Numeric identifier columns (`model_year`, `vehicle_id`, ratings, flags) are set to **Don't summarize** so Power BI never adds them up.
- Band columns (`price_band`, `range_band`) are prefixed with 1., 2., 3., 4. in SQL so they sort correctly without extra sort columns.
- One colour per market segment is used consistently across the report.
- Data quality flag columns (`dup_key_flag`, `body_type_conflict_flag`) come through from SQL so questionable rows can be filtered out.

## How to open the report

1. Open `EV_Market_Dashboard.pbix` in Power BI Desktop.
2. Go to **Home > Transform data > Data source settings** and point the SQL Server connection to your own server and the `EV_Market` database.
3. Refresh the data.

To rebuild from scratch, run the SQL scripts first (see the [SQL README](../sql/README.md)), then connect to `dbo.vw_ev_market`.

## Limitations

- The dataset appears to be simulated. Total revenue is shown on a very large scale because it is estimated as price multiplied by simulated annual units for every row, so the absolute totals should not be compared with real market figures.
- Brands with very few listings (Honda 2, Lucid 6, Xiaomi 8) show unreliable averages and a market share close to 0%.
- The 2026 drop in units may reflect how the data was generated and has not been verified against any external source.
- Around 1,400 rows share the same brand, model, year and variant with different specifications. They are kept in the report and can be excluded using the flag columns.

## Possible next steps

- Add a page navigator and a data quality toggle for the flagged rows
- Add year-on-year growth by brand using the `Units YoY %` measure
- Publish the report to Power BI Service
- Rebuild the pipeline with a bronze, silver and gold layer structure

## Author

**Okerio Davis**
LinkedIn: [linkedin.com/in/okeriodavis](https://linkedin.com/in/okeriodavis)

