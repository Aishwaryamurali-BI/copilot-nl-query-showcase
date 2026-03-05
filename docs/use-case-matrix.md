# Copilot Natural Language Query — Use Case Matrix

## Document Information

| Field | Detail |
|-------|--------|
| **Project** | Copilot Natural Language Query Showcase |
| **Dataset** | Logistics Operations — 12,000 Shipments (2023–2024) |
| **Semantic Model** | 7 tables: FactShipments + 6 dimension tables |
| **Purpose** | Map business questions to Copilot capabilities and expected outputs |

---

## How to Read This Matrix

Each row represents a real business question that a logistics manager, operations analyst, or executive might ask. The matrix shows exactly how Copilot interprets the query, which tables and measures are involved, and what the user should expect to see as a result.

The **Complexity Rating** indicates how likely Copilot is to answer correctly on the first attempt:

- 🟢 **Simple** — Copilot handles these reliably (90%+ accuracy)
- 🟡 **Moderate** — Copilot usually gets it right but may need a follow-up prompt
- 🔴 **Complex** — Copilot may struggle; a pre-built report is often better

---

## Volume & Count Queries

| # | Natural Language Query | Tables Used | Measures / Columns Involved | Expected Answer Format | Expected Result | Complexity | Business Value |
|---|----------------------|-------------|----------------------------|----------------------|----------------|------------|---------------|
| 1 | "How many shipments did we have last month?" | FactShipments, DimDate | Total Shipments measure, DimDate[MonthName] | Single KPI number | ~500 shipments (varies by month) | 🟢 Simple | Quick volume check for managers during stand-ups |
| 2 | "Total shipments by year" | FactShipments, DimDate | Total Shipments, DimDate[Year] | Bar chart or table with 2 rows | 2023: ~6,052 / 2024: ~5,948 | 🟢 Simple | Year-over-year volume comparison |
| 3 | "How many packages did we ship this quarter?" | FactShipments, DimDate | Total Packages measure, DimDate[Quarter] | Single KPI number | SUM of PackageCount for latest quarter | 🟢 Simple | Quarterly operational reporting |
| 4 | "Show shipment count by product category" | FactShipments, DimProductCategory | Total Shipments, CategoryName | Bar chart with 8 bars | ~1,500 per category (evenly distributed) | 🟢 Simple | Category demand analysis |

## Delivery Performance Queries

| # | Natural Language Query | Tables Used | Measures / Columns Involved | Expected Answer Format | Expected Result | Complexity | Business Value |
|---|----------------------|-------------|----------------------------|----------------------|----------------|------------|---------------|
| 5 | "What is our on-time delivery rate?" | FactShipments | On-Time Delivery Rate measure | Single KPI percentage | ~31.0% | 🟢 Simple | Core KPI monitoring — immediate visibility into service levels |
| 6 | "Show delivery performance by warehouse" | FactShipments, DimWarehouse | Average Delivery Days, WarehouseName | Table or bar chart with 6 rows | Avg days per warehouse (range: 5.5–7.8 days) | 🟢 Simple | Identify which warehouses are under/over-performing |
| 7 | "Which ship mode has the best on-time rate?" | FactShipments, DimShipMode | On-Time Delivery Rate, ShipMode | Ranked table with 6 rows | Two-Day Air leads at ~31.7%, Freight (LTL) lowest at ~29.7% | 🟡 Moderate | Helps logistics planners choose optimal shipping methods |
| 8 | "Average delivery days for Next Day Air" | FactShipments, DimShipMode | Average Delivery Days, ShipMode filtered | Single number | ~2.9 days (target: 1 day) | 🟢 Simple | SLA compliance check for premium shipping tier |
| 9 | "Compare delivery days: target vs actual by ship mode" | FactShipments, DimShipMode | Average Delivery Days, TargetDays, ShipMode | Table with target and actual columns side by side | All modes exceed targets by 1.8–2.0 days on average | 🟡 Moderate | Gap analysis for operational improvement planning |

## Cost Analysis Queries

| # | Natural Language Query | Tables Used | Measures / Columns Involved | Expected Answer Format | Expected Result | Complexity | Business Value |
|---|----------------------|-------------|----------------------------|----------------------|----------------|------------|---------------|
| 10 | "Which carrier has the lowest cost?" | FactShipments, DimCarrier | Cost per Shipment measure, CarrierName | Ranked table or bar chart | Carriers ranked by avg cost per shipment | 🟢 Simple | Direct cost optimization — renegotiate contracts with expensive carriers |
| 11 | "Total shipping cost this year" | FactShipments, DimDate | Total Shipping Cost, DimDate[Year] | Single dollar amount | ~$580K for 2024 | 🟢 Simple | Budget tracking |
| 12 | "Compare air vs ground shipping cost" | FactShipments, DimCarrier | Average Shipping Cost, CarrierType | Comparison chart or table | Air: ~$150 avg / Ground: ~$65 avg / Rail: ~$45 avg / Ocean: ~$38 avg | 🟡 Moderate | Strategic mode selection — justify when air premium is worth the speed |
| 13 | "Monthly shipping cost trend" | FactShipments, DimDate | Total Shipping Cost, MonthName | Line chart with 24 data points | Range: $38K–$55K per month with seasonal variation | 🟢 Simple | Budget forecasting and seasonal planning |
| 14 | "Cost per shipment by product category" | FactShipments, DimProductCategory | Cost per Shipment, CategoryName | Bar chart with 8 bars | Heavy items (Furniture, Industrial) cost 2–3x more than light items | 🟡 Moderate | Category-level cost allocation for pricing decisions |

## Quality & Satisfaction Queries

| # | Natural Language Query | Tables Used | Measures / Columns Involved | Expected Answer Format | Expected Result | Complexity | Business Value |
|---|----------------------|-------------|----------------------------|----------------------|----------------|------------|---------------|
| 15 | "What is the damage rate for electronics?" | FactShipments, DimProductCategory | Damage Rate measure, CategoryName filtered | Single percentage | ~3% damage rate for Electronics (flagged as "Fragile" handling) | 🟡 Moderate | Quality monitoring — identifies categories needing better packaging |
| 16 | "Show return rate by carrier" | FactShipments, DimCarrier | Return Rate measure, CarrierName | Bar chart with 8 bars | ~5% average return rate, slight variation by carrier | 🟡 Moderate | Carrier accountability — correlate returns with carrier performance |
| 17 | "Average satisfaction score" | FactShipments | Average Satisfaction Score measure | Single number | 3.21 out of 5.0 | 🟢 Simple | Overall service quality pulse check |
| 18 | "Satisfaction score trend over time" | FactShipments, DimDate | Average Satisfaction Score, MonthName | Line chart | Monthly trend with correlation to on-time performance | 🟡 Moderate | Long-term service quality tracking — board-level reporting |
| 19 | "Which warehouse has the highest damage rate?" | FactShipments, DimWarehouse | Damage Rate, WarehouseName | Ranked table | Identifies problem warehouses for operational review | 🟡 Moderate | Targeted warehouse improvement initiatives |

## Regional & Capacity Queries

| # | Natural Language Query | Tables Used | Measures / Columns Involved | Expected Answer Format | Expected Result | Complexity | Business Value |
|---|----------------------|-------------|----------------------------|----------------------|----------------|------------|---------------|
| 20 | "Total weight shipped to West region" | FactShipments, DimDestination | Total Weight Shipped measure, Region filtered | Single number in lbs | Sum of TotalWeightLbs where Region = "West" | 🟢 Simple | Capacity planning for regional distribution centers |
| 21 | "Shipments by destination region" | FactShipments, DimDestination | Total Shipments, Region | Bar chart with 4 bars | East, West, Central, South distribution | 🟢 Simple | Regional demand balancing |
| 22 | "Top 5 destination cities by shipment volume" | FactShipments, DimDestination | Total Shipments, City with Top N filter | Ranked table with 5 rows | Top cities ranked by volume | 🟡 Moderate | Last-mile delivery optimization |
| 23 | "Show warehouse utilization by region" | FactShipments, DimWarehouse | Total Shipments, Total Weight, WarehouseName, Region | Matrix or grouped table | Shipments and weight per warehouse, grouped by region | 🔴 Complex | Infrastructure investment decisions |

## Multi-Dimensional & Advanced Queries

| # | Natural Language Query | Tables Used | Measures / Columns Involved | Expected Answer Format | Expected Result | Complexity | Business Value |
|---|----------------------|-------------|----------------------------|----------------------|----------------|------------|---------------|
| 24 | "Which carrier delivers fastest for heavy items?" | FactShipments, DimCarrier, DimProductCategory | Average Delivery Days, CarrierName, WeightClass filtered | Filtered and ranked table | Best carrier for heavy shipments specifically | 🔴 Complex | Carrier assignment rules for heavy-weight orders |
| 25 | "Show me cost per shipment trend, split by carrier type" | FactShipments, DimCarrier, DimDate | Cost per Shipment, CarrierType, MonthName | Multi-line chart | Air, Ground, Rail, Ocean trends over time | 🔴 Complex | Long-term cost trend analysis by mode |
| 26 | "Is there a correlation between delivery delay and satisfaction score?" | FactShipments | ActualDeliveryDays, CustomerSatisfactionScore | Scatter plot or insight text | Negative correlation — longer delays = lower scores | 🔴 Complex | Quantifies business impact of delivery delays |

---

## Summary Statistics

| Metric | Value |
|--------|-------|
| Total use cases documented | 26 |
| 🟢 Simple (Copilot reliable) | 12 (46%) |
| 🟡 Moderate (Copilot usually works) | 10 (38%) |
| 🔴 Complex (pre-built report recommended) | 4 (15%) |
| Tables referenced | All 7 tables in the model |
| Unique measures used | 10 measures |

---

## Recommended Copilot Demo Script

For a live demonstration, execute these queries in sequence to tell a compelling story:

1. **Start broad:** "How many shipments did we have this year?" (establishes scale)
2. **Check the key metric:** "What is our on-time delivery rate?" (reveals the 31% problem)
3. **Drill into the problem:** "Show on-time rate by carrier" (identifies which carriers lag)
4. **Explore cost trade-offs:** "Compare air vs ground shipping cost" (cost vs. speed)
5. **Check quality:** "What is our damage rate?" (identifies handling issues)
6. **Find trends:** "Monthly shipping cost trend" (shows seasonality)
7. **Get regional insight:** "Total weight shipped by region" (capacity planning)
8. **End with satisfaction:** "Satisfaction score trend over time" (ties operations to customer experience)

This 8-query sequence takes approximately 5 minutes and demonstrates breadth across volume, performance, cost, quality, and customer satisfaction — the five pillars of logistics operations.

---

*This matrix should be updated as new measures are added to the semantic model or as Copilot capabilities evolve with Power BI updates.*
