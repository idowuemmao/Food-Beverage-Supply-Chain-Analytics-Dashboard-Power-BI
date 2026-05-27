# 🏭 Supply Chain Intelligence Report — Power BI

> **End-to-end supply chain performance monitoring for food & beverage operations.**
> Built for supply chain managers and executives to move from high-level KPIs to root cause analysis in a single report.

---

![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-F2C811?style=flat-square&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-Measures%20%26%20Calculated%20Columns-0078D4?style=flat-square)
![Pages](https://img.shields.io/badge/Report%20Pages-4-4CAF50?style=flat-square)
![Status](https://img.shields.io/badge/Status-Complete-success?style=flat-square)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Report Structure](#report-structure)
- [Page 1 — Executive Overview](#page-1--executive-overview)
- [Page 2 — Operations & Inventory Performance](#page-2--operations--inventory-performance)
- [Page 3 — Channel Deep Dive](#page-3--channel-deep-dive)
- [Page 4 — Supplier Deep Dive](#page-4--supplier-deep-dive)
- [Data Model](#data-model)
- [DAX Measures Reference](#dax-measures-reference)
- [Calculated Columns](#calculated-columns)
- [Navigation & Interactivity](#navigation--interactivity)
- [How to Use](#how-to-use)
- [Business Questions Answered](#business-questions-answered)
- [Design Principles](#design-principles)

---

## Overview

This Power BI report covers end-to-end supply chain operations for a food and beverage distribution network. It spans the full operational lifecycle — from order placement through supplier fulfilment, warehouse management, channel delivery, and customer profitability — surfacing the financial impact of every operational failure along the way.

The report is structured as a **progressive disclosure model**: Page 1 gives executives the headline picture, Page 2 exposes operational risk, and Pages 3–4 are drill-through destinations for deep channel and supplier analysis.

### Dataset scope

| Table | Description |
|---|---|
| `Fact_Orders` | Transactional order data including revenue, COGS, OTIF, delays, returns, waste, and quality flags |
| `Inventory_Snapshots` | Monthly inventory positions by warehouse and product, including expiry and stockout flags |
| `Dim_Product` | Product catalogue with shelf life, storage type, and standard cost |
| `Dim_Supplier` | Supplier profiles including risk tier, reliability %, planned lead time, and defect rate |
| `Dim_Customer` | Customer master with segment, region, and priority |
| `Dim_Warehouse` | Warehouse locations, capacity, and cold storage capability |
| `Dim_Date` | Full date dimension with fiscal flags (IsSummer, IsDecemberPeak, IsWeekend) |
| `Dim_Channel` | Channel dimension (Retail, E-commerce, HoReCa, Distributors) |

---

## Report Structure

```
Page 1  ──►  Executive Overview          (KPIs + trends + channel + supplier risk)
   │
   ▼
Page 2  ──►  Operations & Inventory      (Delays + waste + warehouse risk + supplier matrix)
   │
   ├──►  Page 3  ──►  Channel Deep Dive  (Drill-through from channel row)
   │
   └──►  Page 4  ──►  Supplier Deep Dive (Drill-through from supplier row)
```

Pages 3 and 4 are **drill-through pages** — they receive filter context from Pages 1 and 2 and expose granular performance detail for a single channel or supplier.

---

## Page 1 — Executive Overview

**Purpose:** Give supply chain executives a single-screen summary of network health, financial performance, and top-level risk signals.

<img width="889" height="483" alt="page 1" src="https://github.com/user-attachments/assets/57227041-c04b-49b9-9538-e9374a6c6f0b" />

### KPI Cards

| KPI | Measure | Context |
|---|---|---|
| Total Revenue | `SUM(Fact_Orders[Revenue])` | YoY % change subtitle |
| Gross Profit | `SUM(Fact_Orders[GrossProfit])` | GP margin % subtitle |
| OTIF % | `DIVIDE(COUNTROWS(FILTER(...OTIF_Flag=1)), COUNTROWS(...))` | Delta vs 89% target |
| Late Deliveries | `COUNTROWS(FILTER(...LateDays > 0))` | Avg late days subtitle |
| Stockout Count | `COUNTROWS(FILTER(...StockoutFlag=1))` | YoY change subtitle |
| Operational Loss | `[Total Operational Loss $]` | As % of revenue subtitle |

### Visuals

**1. Revenue, Profit & Delivery Reliability Trend** *(Line + area combo)*
- X-axis: `Dim_Date[MonthName]` by year
- Series: `[Total Revenue]`, `[Gross Profit]`, `[OTIF %]` on secondary axis
- Reveals seasonality, margin compression, and delivery degradation on one canvas

**2. Channel Profitability Performance** *(Clustered column — drill-through enabled)*
- X-axis: `Dim_Channel[Channel]`
- Values: `[Total Revenue]`, `[Gross Profit]`, `[GP Margin %]` as data labels
- Right-click any channel bar → drill through to Page 3

**3. Operational Risk & Service Performance Matrix** *(Matrix table)*
- Rows: `Dim_Channel[Channel]`
- Columns: Revenue, OTIF %, Stockout Orders, Return Rate %, Late Deliveries, Avg Late Days, Quality Issues, Quality Issue Rate %, Operational Loss
- Conditional formatting: red/amber/green scales on OTIF %, Return Rate %, Operational Loss

**4. Supplier Risk & Reliability Analysis** *(Scatter plot)*
- X-axis: `Dim_Supplier[ReliabilityPct]`
- Y-axis: `[Avg Defect Rate %]`
- Bubble label: `Dim_Supplier[SupplierName]`
- Legend: `Dim_Supplier[RiskTier]` (High = red, Medium = amber, Low = green)
- Top-left quadrant = immediate action zone

---

## Page 2 — Operations & Inventory Performance

**Purpose:** Root cause analysis of delivery failures, inventory risk, waste, and supplier quality. The analytical bridge between the executive summary and the drill-through detail pages.

<img width="888" height="502" alt="page 2" src="https://github.com/user-attachments/assets/98d1e57a-32d8-4fe6-93c3-fe636423727f" />

### KPI Cards

| KPI | Measure | Context |
|---|---|---|
| Total Return Qty | `SUM(Fact_Orders[ReturnQty])` | Return rate % subtitle |
| Total Waste Qty | `SUM(Fact_Orders[WasteQty])` | Cost $ subtitle |
| Quality Issue Rate | `DIVIDE(COUNTROWS(FILTER(...QualityIssueFlag=1)), COUNTROWS(...))` | Count of affected orders |
| Avg Actual Lead Days | `AVERAGEX(Fact_Orders, DATEDIFF([OrderDate],[ShipDate],DAY))` | vs planned lead time |
| Total Expired Qty | `SUM(Inventory_Snapshots[ExpiredQty])` | As % of shipped qty |
| Inventory at Risk | Closing stock below reorder point | % of closing stock |

### Visuals

**1. Delivery Delay Distribution** *(Pie / donut chart)*
- Values: `COUNTROWS(Fact_Orders)` grouped by `[LateBand]` calculated column
- LateBand = On-Time / 1–3 days / 3.5–5 days / 5+ days
- Shows the severity profile of late deliveries — not just the count

**2. Warehouse Inventory Risk Exposure** *(Stacked column chart)*
- X-axis: `Dim_Warehouse[WarehouseName]`
- Series: `[Total Expired Qty]`, `[Inventory at Risk]`
- Identifies which warehouses carry the highest perishable risk

**3. Returns & Waste by Category → Subcategory → Product** *(Clustered column — drill-down)*
- X-axis: `Dim_Product[Category]` → `Subcategory` → `ProductName`
- Series: `[Total Waste Qty]`, `[Total Return Qty]`
- Three-level drill-down reveals the specific SKUs driving losses

**4. Supplier Reliability vs Defect Rate Matrix** *(Matrix table — drill-through enabled)*
- Rows: `Dim_Supplier[SupplierName]`
- Columns: GM %, OTIF %, Avg Late Days, Late Deliveries, Quality Issues, Lead Time Gap, GP Erosion, Stockout %, Operational Loss
- Right-click any supplier row → drill through to Page 4
- Conditional formatting on all metric columns

**5. Warehouse Stockout Heatmap** *(Matrix table)*
- Rows: `Dim_Warehouse[WarehouseName]`
- Columns: `Dim_Product[Category]`
- Values: `[Stockout Rate %]`
- Background color scale: green (0%) → red (25%+)
- Pinpoints which warehouse × category combinations are chronic stockout risks

---

## Page 3 — Channel Deep Dive

**Purpose:** Granular performance analysis for a single channel. Activated via drill-through from Page 1 or Page 2. All visuals respond to the channel context filter passed from the source page.

<img width="889" height="499" alt="page 3" src="https://github.com/user-attachments/assets/ed023545-b8a5-411c-845a-9273b1d56b46" />

### KPI Cards

| KPI | Measure |
|---|---|
| Revenue | `[Total Revenue]` |
| Gross Profit | `[Gross Profit]` with margin % |
| OTIF % | `[OTIF %]` with delta vs 89% target |
| Return Rate % | `[Return Rate %]` — highest subcategory in subtitle |
| Avg Discount % | `AVERAGE(Fact_Orders[DiscountPct])` — promo orders noted |
| Operational Loss | `[Total Operational Loss $]` — as % of revenue |

### Slicers
Storage Type · Brand · Promo Flag · Supplier Name · Priority

### Visuals

**1. Revenue & Profitability Trend** *(Line + area — drill-down)*
- X-axis: `Dim_Date[Year]` → `Quarter` → `Month`
- Series: `[Total Revenue]`, `[Total COGS]`, `[Gross Profit]`
- Shows whether the channel's margin profile is improving or deteriorating over time

**2. Warehouse Service Performance** *(Clustered column)*
- X-axis: `Dim_Warehouse[WarehouseName]`
- Series: `[OTIF %]`, `[GP Margin %]`, `[Return Rate %]`
- Identifies which fulfilment nodes are underserving the channel

**3. Returns vs Profit Analysis** *(Scatter plot — drill-down)*
- X-axis: `[Total Return Qty]`
- Y-axis: `[Gross Profit]`
- Dimension: `Dim_Product[Category]` → `Subcategory` → `ProductName`
- Bubble size: `[Total Revenue]`
- Surfaces the products with high returns and strong profitability — the highest-priority recovery targets

**4. Customer & Channel Performance Matrix** *(Table — category filter buttons)*
- Rows: `Dim_Customer[CustomerName]`
- Columns: Sales Region, Priority, Revenue, Gross Margin, Margin %, OTIF %, Avg Late Days, Total Orders, Stockout Orders, Late Deliveries, Return Qty, Return Rate %, Lead Time Gap, Operational Loss, GP Erosion %
- Category filter buttons (Convenience / Distributor / E-commerce / HoReCa / Retail Chain) switch the customer view using bookmarks
- Conditional formatting on OTIF %, Return Rate %, Operational Loss

---

## Page 4 — Supplier Deep Dive

**Purpose:** Single-supplier performance forensics. Activated via drill-through from the supplier matrix on Page 2. Covers delivery reliability, lead time gaps, product-level profitability, and waste.

<img width="889" height="501" alt="page 4" src="https://github.com/user-attachments/assets/a15818d5-e2d7-48b7-99e3-2fe20b021a31" />

### KPI Cards

| KPI | Measure |
|---|---|
| Supplier OTIF % | `CALCULATE([OTIF %], supplier context)` |
| Actual Lead Time | `[Avg Actual Lead Time]` vs planned in subtitle |
| Defect Rate % | `RELATED(Dim_Supplier[AvgDefectRate])` |
| Supplier Revenue | `[Total Revenue]` across all channels |
| Gross Profit Erosion | `[Total Operational Loss $]` — largest driver in subtitle |
| GP Erosion % | `[Operational Loss % of GP]` — acceptable band noted |

### Context banner
Displays the selected supplier's Risk Tier, City/Country, Reliability %, and Supplier Region — pulled automatically from `Dim_Supplier` via the drill-through filter context.

### Slicers
Channel · Brand · Promo Flag · Supplier Name · Priority

### Visuals

**1. Supplier OTIF vs Reliability Target** *(Line chart — drill-down)*
- X-axis: `Dim_Date[Quarter]` → `Month`
- Series: `[OTIF %]` (line), `Dim_Supplier[ReliabilityPct]` (constant reference line)
- Shows how actual delivery performance tracks against the supplier's own contracted reliability target over time

**2. Planned vs Actual Lead Time** *(Clustered column + line — drill-down)*
- X-axis: `Dim_Date[Quarter]` → `Month`
- Bar series: `Dim_Supplier[PlannedLeadTimeDays]`, `[Avg Actual Lead Time]`
- Line (secondary axis): `[Lead Time Gap]`
- Quantifies cumulative schedule slippage per reporting period

**3. Supplier Product Performance Matrix** *(Table)*
- Rows: `Dim_Product[ProductName]`
- Columns: Brand, Storage Type, Revenue, Gross Margin, Margin %, OTIF %, GP Erosion, Avg Late Days, Total Orders, Late Deliveries, Quality Issues, Return Qty, Waste Qty, Lead Time Gap, Operational Loss
- Category summary headers (Bakery / Beverages / Dairy / Frozen / Snacks) with subtotals
- Conditional formatting on OTIF %, GP Erosion, Lead Time Gap
- Grand total row pinned to bottom

---

## Data Model

```
Dim_Date ──────────────────┐
Dim_Channel ───────────────┤
Dim_Customer ──────────────┼──── Fact_Orders ────┬──── Dim_Product ──── (Category, Subcategory)
Dim_Supplier ──────────────┤                     │
Dim_Warehouse ─────────────┘                     └──── Inventory_Snapshots
```

All dimension tables connect to `Fact_Orders` on their respective ID keys using single-direction (→) active relationships. `Inventory_Snapshots` connects to both `Dim_Product` and `Dim_Warehouse` independently.

---

## DAX Measures Reference

### Financial

```dax
Total Revenue = SUM(Fact_Orders[Revenue])

Gross Profit = SUM(Fact_Orders[GrossProfit])

GP Margin % = DIVIDE([Gross Profit], [Total Revenue], 0)

Total COGS = SUM(Fact_Orders[COGS])

Revenue YoY % =
    VAR CY = [Total Revenue]
    VAR PY = CALCULATE([Total Revenue], SAMEPERIODLASTYEAR(Dim_Date[Date]))
    RETURN DIVIDE(CY - PY, PY, 0)
```

### Delivery & OTIF

```dax
OTIF % =
DIVIDE(
    COUNTROWS(FILTER(Fact_Orders, Fact_Orders[OTIF_Flag] = 1)),
    COUNTROWS(Fact_Orders),
    0
)

Late Deliveries =
COUNTROWS(FILTER(Fact_Orders, Fact_Orders[LateDays] > 0))

Late Delivery % =
DIVIDE(
    COUNTROWS(FILTER(Fact_Orders, Fact_Orders[LateDays] > 0)),
    COUNTROWS(Fact_Orders),
    0
)

Avg Late Days =
AVERAGEX(
    FILTER(Fact_Orders, Fact_Orders[LateDays] > 0),
    Fact_Orders[LateDays]
)

Avg Actual Lead Time =
AVERAGEX(Fact_Orders, DATEDIFF(Fact_Orders[OrderDate], Fact_Orders[ShipDate], DAY))

Lead Time Gap =
[Avg Actual Lead Time] - AVERAGE(Dim_Supplier[PlannedLeadTimeDays])
```

### Inventory & Quality

```dax
Stockout Count =
COUNTROWS(FILTER(Fact_Orders, Fact_Orders[StockoutFlag] = 1))

Stockout Rate % =
DIVIDE([Stockout Count], COUNTROWS(Fact_Orders), 0)

Total Return Qty = SUM(Fact_Orders[ReturnQty])

Return Rate % =
DIVIDE(SUM(Fact_Orders[ReturnQty]), SUM(Fact_Orders[OrderQty]), 0)

Total Waste Qty = SUM(Fact_Orders[WasteQty])

Waste Cost $ =
SUMX(Fact_Orders, Fact_Orders[WasteQty] * Fact_Orders[UnitCost])

Quality Issue Rate =
DIVIDE(
    COUNTROWS(FILTER(Fact_Orders, Fact_Orders[QualityIssueFlag] = 1)),
    COUNTROWS(Fact_Orders),
    0
)

Total Expired Qty = SUM(Inventory_Snapshots[ExpiredQty])

Inventory at Risk =
COUNTROWS(
    FILTER(
        Inventory_Snapshots,
        Inventory_Snapshots[ClosingStock] < Inventory_Snapshots[ReorderPoint]
    )
)
```

### Operational Loss Framework

```dax
-- Hard cost losses (cash spent, zero revenue return)
Waste Loss $ =
SUMX(Fact_Orders, Fact_Orders[WasteQty] * Fact_Orders[UnitCost])

Return Loss $ =
SUMX(
    Fact_Orders,
    Fact_Orders[ReturnQty]
        * (Fact_Orders[UnitPrice] * (1 - Fact_Orders[DiscountPct])
            - Fact_Orders[UnitCost])
)

-- Opportunity losses (GP foregone due to operational failure)
Stockout Loss $ =
CALCULATE(SUM(Fact_Orders[GrossProfit]), Fact_Orders[StockoutFlag] = 1)

Late Delivery Loss $ =
CALCULATE(SUM(Fact_Orders[GrossProfit]), Fact_Orders[OTIF_Flag] = 0)

Quality Issue Loss $ =
CALCULATE(SUM(Fact_Orders[GrossProfit]), Fact_Orders[QualityIssueFlag] = 1)

-- Subtotals
Hard Cost Loss $ = [Waste Loss $] + [Return Loss $]

Opportunity Loss $ =
    [Stockout Loss $] + [Late Delivery Loss $] + [Quality Issue Loss $]

-- Consolidated total
Total Operational Loss $ = [Hard Cost Loss $] + [Opportunity Loss $]

-- Ratios
Operational Loss % of GP =
DIVIDE(
    [Total Operational Loss $],
    SUM(Fact_Orders[GrossProfit]) + [Total Operational Loss $],
    0
)

Operational Loss Rate % =
DIVIDE([Total Operational Loss $], SUM(Fact_Orders[Revenue]), 0)

Adjusted GP $ =
SUM(Fact_Orders[GrossProfit]) - [Total Operational Loss $]

Avg Operational Loss per Order =
DIVIDE([Total Operational Loss $], COUNTROWS(Fact_Orders), 0)
```

### Supplier & Promo

```dax
Supplier OTIF % =
CALCULATE(
    [OTIF %],
    USERELATIONSHIP(Fact_Orders[SupplierID], Dim_Supplier[SupplierID])
)

Supplier Loss Index =
DIVIDE([Total Operational Loss $], COUNTROWS(Fact_Orders), 0)

Promo Revenue = CALCULATE([Total Revenue], Fact_Orders[PromoFlag] = 1)

Promo OTIF % = CALCULATE([OTIF %], Fact_Orders[PromoFlag] = 1)

Promo GP Margin % = CALCULATE([GP Margin %], Fact_Orders[PromoFlag] = 1)

Promo Return Rate % = CALCULATE([Return Rate %], Fact_Orders[PromoFlag] = 1)

Promo Share % = DIVIDE([Promo Revenue], [Total Revenue], 0)
```

---

## Calculated Columns

```dax
-- Fact_Orders
LateBand =
IF(
    Fact_Orders[LateDays] = 0, "On-Time",
    IF(Fact_Orders[LateDays] <= 3, "1–3 days",
    IF(Fact_Orders[LateDays] <= 5, "3.5–5 days", "5+ days"))
)

ActualLeadTimeDays =
DATEDIFF(Fact_Orders[OrderDate], Fact_Orders[ShipDate], DAY)

OrderMonth = FORMAT(Fact_Orders[OrderDate], "YYYY-MM")

-- Dim_Product
ShelfLifeRiskFlag =
IF(Dim_Product[ShelfLifeDays] < 14, "Short-Life", "Standard")

-- Dim_Supplier
RiskTierColor =
SWITCH(
    Dim_Supplier[RiskTier],
    "High",   "Red",
    "Medium", "Amber",
    "Low",    "Green",
    "Grey"
)
```

---

## Navigation & Interactivity

| Interaction | Behaviour |
|---|---|
| Page 1 channel bar → right-click | Drill through to Page 3 (Channel Deep Dive) |
| Page 2 supplier matrix row → right-click | Drill through to Page 4 (Supplier Deep Dive) |
| Pages 3 & 4 "Go Back" button | Returns to the source page with context preserved |
| Page 3 category filter buttons | Switch customer matrix via bookmarks — no slicer required |
| All scatter plots | Tooltip shows entity name, risk tier, and key metrics on hover |
| All matrix visuals | Conditional formatting encodes performance severity visually |
| All drill-down visuals | Hierarchy enabled: Category → Subcategory → Product |
| Reset Page button | Clears all active slicer selections and resets to full dataset |

---

## How to Use

### Prerequisites
- Power BI Desktop (latest version recommended)
- Access to the source dataset (Excel / SQL / CSV)

### Setup

1. Clone or download this repository
2. Open `SupplyChainReport.pbix` in Power BI Desktop
3. In **Transform Data**, update the data source connection to your dataset path
4. Refresh the data model
5. Publish to Power BI Service to share with stakeholders

### Drill-through

Right-click any **channel row** in the Page 1 matrix or any **supplier row** in the Page 2 supplier matrix. Select the relevant drill-through option from the context menu. The destination page loads pre-filtered to that entity. Use the **Go Back** button (top-left of Pages 3 and 4) to return to the source page.

### Reset filters

Each page includes a **Reset Page** button in the bottom-left panel. Click it to clear all active slicer selections and return to the unfiltered dataset view.

---

## Business Questions Answered

| Question | Page | Visual |
|---|---|---|
| How is overall supply chain performance trending? | 1 | Revenue, Profit & Delivery Reliability Trend |
| Which channel is most and least profitable? | 1 | Channel Profitability Performance |
| Which channels have the worst OTIF and operational loss? | 1 | Operational Risk & Service Performance Matrix |
| Which suppliers are high-risk? | 1 | Supplier Risk & Reliability Scatter |
| Where are delivery delays concentrated by severity? | 2 | Delivery Delay Distribution |
| Which warehouses have the most inventory risk? | 2 | Warehouse Inventory Risk Exposure |
| Which SKUs drive the most waste and returns? | 2 | Returns & Waste by Category drill-down |
| Which suppliers are underperforming operationally? | 2 | Supplier Reliability vs Defect Rate Matrix |
| Which warehouse × category combinations have chronic stockouts? | 2 | Warehouse Stockout Heatmap |
| How does a specific channel's profitability trend over time? | 3 | Revenue & Profitability Trend |
| Which warehouses are failing a specific channel? | 3 | Warehouse Service Performance |
| Which products in a channel have high returns against strong margins? | 3 | Returns vs Profit Analysis Scatter |
| Which customers within a channel are high-risk or low-margin? | 3 | Customer & Channel Performance Matrix |
| Is a specific supplier meeting their contracted lead time? | 4 | Planned vs Actual Lead Time |
| Is a supplier's OTIF tracking against their reliability target? | 4 | Supplier OTIF vs Reliability Target |
| Which products from a supplier are causing the most GP erosion? | 4 | Supplier Product Performance Matrix |

---

## Design Principles

**Executive-first layout.** Page 1 is readable in under 60 seconds. Every KPI card includes a contextual subtitle so numbers tell their own story without additional explanation.

**Progressive disclosure.** Each page layer adds depth without replacing the previous view. Executives work on Page 1; analysts move through Pages 2–4.

**Financial grounding.** Every operational metric connects to a financial outcome. OTIF %, Late Days, and Stockout Count are always paired with their GP erosion equivalent, enabling prioritisation by financial impact rather than event volume.

**Consistent conditional formatting.** Red / amber / green logic applies uniformly across all matrix and table visuals using identical thresholds, giving users an immediate visual vocabulary for risk across all pages.

**Drill-through over page proliferation.** Rather than creating separate pages for each channel or supplier, drill-through pages receive filter context from the source, keeping the report concise while enabling unlimited analytical depth.

---

*Built with Power BI Desktop · DAX · Food & Beverage Supply Chain Dataset*
