# Project 4: Customer Segmentation (RFM Analysis)

**Goal:** segment customers by **R**ecency, **F**requency and **M**onetary value so marketing and sales can target the right groups.

**Data:** 5,000 orders from 500 customers (January 2023 to December 2024), joined to Products and Salespersons dimensions.

## How the RFM model is built
A customer-level table is created in DAX, then scored with `SWITCH` rules.

```dax
Customer RFM =
ADDCOLUMNS (
    Customers,
    "Last Purchase", CALCULATE ( MAX ( Sales[Date] ) ),
    "Frequency",     CALCULATE ( DISTINCTCOUNT ( Sales[Order ID] ) ),
    "Monetary",      CALCULATE ( SUMX ( Sales, Sales[Quantity] * Sales[Unit Price] ) )
)

Recency =
VAR MaxDate = CALCULATE ( MAX ( Sales[Date] ), ALL ( Sales ) )
RETURN DATEDIFF ( [Last Purchase], MaxDate, DAY )
```

**Scoring bands (5 = best):**

| Score | Recency (days since last order) | Frequency (orders) | Monetary (total spend) |
|---|---|---|---|
| 5 | ≤ 30 | ≥ 15 | ≥ 1,800,000 |
| 4 | ≤ 60 | ≥ 12 | ≥ 1,200,000 |
| 3 | ≤ 120 | ≥ 9 | ≥ 800,000 |
| 2 | ≤ 180 | ≥ 6 | ≥ 400,000 |
| 1 | > 180 | < 6 | < 400,000 |

**Segment rules (evaluated in this order):**
```dax
Customer Segment =
SWITCH ( TRUE (),
    [Recency Score] >= 4 && [Frequency Score] >= 4 && [Monetary Score] >= 4, "Champions",
    [Frequency Score] >= 4,                                                  "Loyal Customers",
    [Monetary Score] >= 4,                                                   "Big Spenders",
    [Recency Score] <= 2 && [Frequency Score] <= 2,                          "Lost Customers",
    "Regular Customers" )
```
Measures: `Revenue`, `Total Orders`, `Customer Count`, `Revenue per Customer`.

## Dashboard
KPI cards (customers, revenue, revenue per customer, orders); segment mix (donut); revenue by segment and by segment and region; RFM scatter plot (frequency vs monetary by segment); Top Customers table; slicers for segment, city and region.

## Results

| Segment | Customers | % of customers | Revenue | % of revenue |
|---|---|---|---|---|
| Big Spenders | 168 | 33.6% | 304.3M | 40.2% |
| Champions | 95 | 19.0% | 218.1M | 28.8% |
| Regular Customers | 149 | 29.8% | 117.5M | 15.5% |
| Loyal Customers | 58 | 11.6% | 94.3M | 12.4% |
| Lost Customers | 30 | 6.0% | 23.4M | 3.1% |

Champions are 19% of customers but generate 28.8% of revenue, and Lost Customers account for only 3.1%.

## Files
- `Customer_Segmentation_RFM.pbix`
