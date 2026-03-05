# Power BI Copilot Evaluation Report

## Document Information

| Field | Detail |
|-------|--------|
| **Project** | Copilot Natural Language Query Showcase |
| **Dataset** | Logistics Operations — 12,000 Shipments across 8 carriers, 6 warehouses, 30 destinations |
| **Evaluation Period** | Testing conducted across 26 documented use cases |
| **Model Configuration** | Star schema with 1 fact table and 6 dimension tables, 12 DAX measures |

---

## 1. Executive Summary

Power BI Copilot enables business users to query data using plain English instead of writing DAX formulas or building reports manually. This evaluation tested Copilot against a logistics semantic model to determine what it handles well, where it struggles, and when organizations should use Copilot versus traditional pre-built reports.

**Key Finding:** Copilot is highly effective for quick, ad-hoc questions involving single aggregations and simple filters (estimated 90%+ accuracy). It becomes progressively less reliable as query complexity increases — particularly with multi-step calculations, multi-table joins with conditional logic, and specific visual formatting requests. The biggest factor in Copilot accuracy is not Copilot itself but the quality of the underlying semantic model.

---

## 2. What Copilot Handles Well

### 2.1 Simple Aggregations

Copilot excels at answering questions that map directly to a single measure or a straightforward SUM, COUNT, or AVERAGE operation.

**Examples that work reliably:**

| Query | Why It Works |
|-------|-------------|
| "Total shipments" | Maps directly to COUNTROWS — single measure, no filter |
| "Average shipping cost" | Maps to AVERAGE(ShippingCost) — single column aggregation |
| "Total weight shipped" | Maps to SUM(TotalWeightLbs) — clear column name |
| "How many carriers do we have?" | Maps to DISTINCTCOUNT — simple count |

**Why these work:** The measure names are descriptive and unambiguous. "Total Shipments" leaves no room for misinterpretation. Copilot recognizes standard aggregation words (total, average, count, sum, maximum, minimum) and maps them to the appropriate DAX function.

### 2.2 Time-Based Filters

Copilot handles temporal filtering well when the model includes a proper date table with standard columns.

**Examples that work reliably:**

| Query | How Copilot Interprets It |
|-------|--------------------------|
| "Shipments last quarter" | Filters DimDate to the previous calendar quarter |
| "Shipping cost in January 2024" | Filters DimDate[MonthName] = "January" AND DimDate[Year] = 2024 |
| "Year-over-year shipment trend" | Groups by Year, shows comparison |
| "Monthly cost trend" | Groups by Month, displays as line chart |

**Why these work:** The DimDate table follows standard naming conventions (Year, Quarter, MonthName, Date) that Copilot is trained to recognize. The date hierarchy is the most well-supported dimension type in Copilot.

### 2.3 Single-Dimension Breakdowns

Copilot reliably groups data by one categorical dimension when the column name clearly describes its contents.

**Examples that work reliably:**

| Query | Dimension Used | Output |
|-------|---------------|--------|
| "Shipments by carrier" | DimCarrier[CarrierName] | Bar chart with 8 bars |
| "Cost by warehouse" | DimWarehouse[WarehouseName] | Table with 6 rows |
| "On-time rate by ship mode" | DimShipMode[ShipMode] | Sorted table |
| "Shipments by region" | DimDestination[Region] | Chart with 4 categories |

**Why these work:** Each dimension table has a single, clearly named descriptive column. "CarrierName" is unambiguous. When Copilot sees "by carrier," it looks for a column containing "carrier" in its name and finds exactly one match.

### 2.4 Simple Filters

Copilot handles single-condition filters well, especially when the filter value matches a recognizable data value.

**Examples that work reliably:**

| Query | Filter Applied |
|-------|---------------|
| "Shipments to the West region" | DimDestination[Region] = "West" |
| "Air cargo shipments only" | DimCarrier[CarrierType] = "Air" |
| "Next Day Air delivery days" | DimShipMode[ShipMode] = "Next Day Air" |
| "Electronics damage rate" | DimProductCategory[CategoryName] = "Electronics" |

---

## 3. Where Copilot Struggles

### 3.1 Multi-Step Calculations

Copilot has difficulty with queries that require computing an intermediate result before producing the final answer.

**Examples that often fail or produce incorrect results:**

| Query | Why It Fails | Correct Approach |
|-------|-------------|-----------------|
| "Which carrier improved their on-time rate the most year-over-year?" | Requires calculating on-time rate per carrier per year, then computing the difference, then ranking | Pre-build a DAX measure: `OT Rate YoY Change` |
| "Shipments where actual delivery exceeded target by more than 5 days" | Requires a row-level comparison between two columns, then counting | Create a calculated column or measure |
| "What percentage of our total cost goes to air shipping?" | Requires calculating air cost as a fraction of total cost — a ratio of two filtered aggregations | Pre-build a `% of Total` measure |

**Root cause:** Copilot generates a single DAX query per request. It does not chain multiple calculation steps together. Each "step" in the user's mental model requires a separate measure or calculated column in the semantic model.

### 3.2 Complex DAX Patterns

Copilot cannot generate advanced DAX patterns that experienced analysts use regularly.

**Patterns Copilot cannot reliably produce:**

| Pattern | Example | Why It Fails |
|---------|---------|-------------|
| Time intelligence beyond basic | "Rolling 3-month average cost" | Requires DATESINPERIOD or DATEADD with window functions |
| CALCULATE with multiple filters | "On-time rate for ground carriers shipping electronics to the West" | Three simultaneous filters across three different tables |
| Iterators (SUMX, AVERAGEX) | "Average cost per pound by category" | Requires row-level division before aggregation |
| SWITCH/IF in measures | "Classify carriers as Premium, Standard, or Economy by their on-time rate" | Conditional logic within a measure definition |
| Semi-additive measures | "End-of-month warehouse inventory levels" | Requires LASTNONBLANK or LASTDATE patterns |

### 3.3 Specific Formatting & Visual Requirements

Copilot chooses the visual type and formatting automatically. Users cannot reliably control the output format.

**Common frustrations:**

| User Expectation | What Copilot Does |
|-----------------|-------------------|
| "Show this as a horizontal bar chart" | May produce a column chart, table, or bar chart — inconsistent |
| "Use red for negative values" | No conditional formatting in Copilot-generated visuals |
| "Show the top 5 only" | May show all values or an incorrect Top N |
| "Format as currency with 2 decimal places" | Relies on the model's format string — does not accept formatting instructions |
| "Put carrier on the Y-axis and cost on X" | Copilot decides axis assignment; explicit axis instructions are often ignored |

### 3.4 Ambiguous Column Names

When the model contains columns with similar or vague names, Copilot frequently picks the wrong one.

**Problem examples from our model:**

| Ambiguity | What Goes Wrong | Fix |
|-----------|----------------|-----|
| "Region" exists in DimDestination AND DimWarehouse | Copilot may filter by warehouse region when user means destination region | Rename to `DestinationRegion` and `WarehouseRegion` |
| "City" exists in DimDestination AND DimWarehouse | Same problem — which city? | Rename to `DestinationCity` and `WarehouseCity` |
| "Cost" could mean ShippingCost, TotalProductCost, or Cost per Shipment | Copilot guesses, sometimes wrong | Use explicit measure names: `Total Shipping Cost`, `Cost per Shipment` |

### 3.5 Negation and Exclusion Queries

Copilot handles inclusion well but often misinterprets exclusion logic.

| Query | Expected Behavior | Common Copilot Error |
|-------|-------------------|---------------------|
| "Shipments NOT to the West region" | Filter to East + Central + South | May still include West, or return no results |
| "Carriers other than SkyLine Air" | Exclude one carrier | May show only SkyLine or ignore the filter |
| "Non-damaged shipments only" | Filter DamageFlag = 0 | May show damaged shipments instead |

---

## 4. Best Practices for Optimizing Copilot Performance

These design choices in the semantic model significantly improve Copilot's accuracy:

### 4.1 Measure Naming

| Principle | Bad Example | Good Example | Why It Matters |
|-----------|-------------|-------------|----------------|
| Use plain business language | `OTD_PCT` | `On-Time Delivery Rate` | Copilot matches user queries to measure names via text similarity |
| Include the aggregation type | `Cost` | `Total Shipping Cost` | Eliminates ambiguity about whether user wants SUM, AVG, or single value |
| Include units where relevant | `Weight` | `Total Weight Shipped (lbs)` | Helps Copilot understand the metric |
| Avoid abbreviations | `Avg Del Days` | `Average Delivery Days` | Copilot understands full words better than abbreviations |

### 4.2 Star Schema Design

| Practice | Benefit for Copilot |
|----------|-------------------|
| One fact table, multiple dimension tables | Copilot navigates relationships cleanly — one path from any dimension to the fact |
| Single-column primary keys | Copilot can join tables unambiguously |
| No snowflake extensions (dimension-to-dimension joins) | Reduces the chance of Copilot choosing the wrong join path |
| Hide technical columns (keys, IDs) from report view | Copilot only sees business-friendly columns, reducing confusion |

### 4.3 Column Naming

| Practice | Impact |
|----------|--------|
| Prefix dimension columns with the entity name when ambiguous | `DestinationRegion` vs. `WarehouseRegion` eliminates confusion |
| Use descriptive names, not codes | `ShipMode` not `SM_CODE` |
| Add descriptions to every column and measure | Copilot reads the Description metadata field to understand context |
| Ensure consistent casing | PascalCase or spaces — pick one pattern and stick with it |

### 4.4 Data Model Hygiene

| Practice | Impact |
|----------|--------|
| Mark a date table as the official date table (Table tools → Mark as date table) | Copilot unlocks full time intelligence capabilities |
| Set default summarization on numeric columns (Sum, Average, None) | Prevents Copilot from choosing the wrong aggregation |
| Create a dedicated measures table (e.g., `_Measures`) | Keeps measures organized and easier for Copilot to discover |
| Hide intermediate calculation measures from report view | Reduces noise — Copilot only sees final business measures |

---

## 5. Recommendation Matrix: Copilot vs. Pre-Built Reports

### 5.1 Decision Framework

| Scenario | Recommended Approach | Rationale |
|----------|---------------------|-----------|
| Ad-hoc question during a meeting | **Copilot** | Speed matters more than formatting. Getting a quick answer in 5 seconds beats waiting for a report to be built. |
| Daily/weekly KPI monitoring | **Pre-built report** | Same questions asked repeatedly should be optimized once as a polished dashboard with conditional formatting, drill-through, and bookmarks. |
| Data exploration / "what if" analysis | **Copilot** | Users don't know what they're looking for yet. Copilot's conversational interface lets them follow threads of inquiry naturally. |
| Board-level or client-facing presentation | **Pre-built report** | Formatting, branding, and visual consistency matter. Copilot cannot produce presentation-quality visuals. |
| New employee onboarding to data | **Copilot** | Asking questions in plain English lowers the barrier to entry. New analysts can explore data without learning DAX. |
| Complex multi-dimensional analysis | **Pre-built report** | Queries involving 3+ dimensions, time intelligence, or conditional logic exceed Copilot's reliable capability. |
| Quick fact-check ("what was our total cost last month?") | **Copilot** | Single-number answers are Copilot's sweet spot. Faster than opening and filtering a report. |
| Operational dashboard refreshed on a schedule | **Pre-built report** | Copilot generates one-time answers; it cannot create persistent, auto-refreshing dashboards. |
| Training data literacy across the organization | **Copilot** | Empowers non-technical users to interact with data. Builds data culture without requiring technical training. |
| Regulatory or audit reporting | **Pre-built report** | Requires exact formatting, consistent definitions, and reproducibility — Copilot's output varies with phrasing. |

### 5.2 Hybrid Approach (Recommended)

The most effective strategy is not choosing one over the other but combining both:

**Layer 1: Pre-built dashboards for the "known questions"**
Build polished reports for the 15–20 questions that stakeholders ask every week. These reports have conditional formatting, drill-through, bookmarks, and scheduled refresh. They are the trusted, certified source of truth.

**Layer 2: Copilot for the "unknown questions"**
Enable Copilot on the same semantic model so users can ask follow-up questions that the pre-built dashboard doesn't answer. "I see the West region is underperforming — which specific city in the West has the worst on-time rate?" This kind of ad-hoc drill-down is where Copilot adds the most value.

**Layer 3: Copilot insights feed report improvements**
Track which questions users ask Copilot repeatedly. If the same question comes up 10+ times, it should become a visual on the pre-built dashboard. Copilot usage analytics become a feedback loop for dashboard design.

---

## 6. Copilot Accuracy Summary

Based on 26 documented use cases tested against the logistics semantic model:

| Complexity Level | Use Cases Tested | Estimated First-Attempt Accuracy | Notes |
|-----------------|-----------------|--------------------------------|-------|
| 🟢 Simple (single aggregation, single filter) | 12 | ~90–95% | Rarely fails when measure names are descriptive |
| 🟡 Moderate (single dimension breakdown with filter, or comparison) | 10 | ~65–75% | May require rephrasing the question or a follow-up prompt |
| 🔴 Complex (multi-dimension, time intelligence, conditional logic) | 4 | ~25–40% | Often produces incorrect results or wrong visual types |

**Overall assessment:** Copilot is a powerful complement to traditional BI, not a replacement. Its value proposition is democratizing data access — enabling the 80% of an organization that does not know DAX to ask questions and get answers. The 20% of power users will continue to need pre-built reports, DAX measures, and full report design capabilities.

---

## 7. Recommendations for This Project

1. **Rename ambiguous columns** — Change `Region` in DimDestination to `DestinationRegion` and in DimWarehouse to `WarehouseRegion` to eliminate Copilot confusion.

2. **Add descriptions to all measures** — In Power BI, select each measure → Properties → Description. Write a one-sentence explanation (e.g., "Counts the total number of shipments in FactShipments"). Copilot uses these descriptions.

3. **Hide technical columns** — In Model view, right-click columns like `DateKey`, `CarrierKey`, `WarehouseKey`, etc. → select "Hide in report view." This prevents Copilot from referencing surrogate keys.

4. **Mark DimDate as the date table** — In Model view, click DimDate → Table tools tab → "Mark as date table" → select the `Date` column. This enables Copilot's full time intelligence support.

5. **Create a measures table** — Consolidate all 12 measures into a `_Measures` table for cleaner organization.

6. **Pre-build a demo report** — Even though this project showcases Copilot, include a traditional dashboard (the one built in Phase C) to demonstrate the hybrid approach in your portfolio.

---

*This evaluation reflects Copilot capabilities as of early 2026. Microsoft is actively improving Copilot with each Power BI monthly update. Re-evaluate quarterly as new features are released.*
