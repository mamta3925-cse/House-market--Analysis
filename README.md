# House-market--Analysis
# Housing Market Analysis Dashboard (Power BI)

# Project Overview
This project analyzes residential real estate transaction data to help stakeholders (real estate agencies, investors, and market analysts) understand pricing trends, regional performance, sales channel behavior, and property-type economics across the housing market.

# Problem Statement
Real estate stakeholders often lack a consolidated view of how **price, region, sales channel, and property type** interact to drive market performance. Without this visibility, it's difficult to answer questions like:
- Which regions are seeing the strongest (or weakest) price growth?
- Are offer prices consistently aligned with purchase prices, or is there systematic negotiation gap?
- Which sales type (auction, family sale, regular sale, other) is underperforming year-over-year?
- Which property types deliver the best price-per-square-meter value and yield?
- What factors (like property age) most influence purchase price?

This dashboard was built in **Power BI** to consolidate housing transaction data into an interactive, multi-page report that answers these questions at a glance.

# Objectives
- Track overall market health via YTD sales, 12-month rolling sales, and units sold
- Compare **median price growth** and **average price per SQM** across regions
- Evaluate the relationship between **offer price and purchase price**
- Break down performance by **sales type** (auction, family sale, regular sale, other sale)
- Compare **property types** (Farm, Villa, Townhouse, Apartment, Summerhouse) on price, inflation/interest/yield, and SQM economics
- Surface **key influencers** driving purchase price changes (e.g., property age)

## 🗂️ Dataset
- Source table: `Housing Data csv`
- Grain: One row per house transaction
- Key fields: `date`, `house_id`, `region`, `city`, `area`, `sales_type` (auction / family_sale / regular_sale / other_sale), `house_type` (Farm / Villa / Townhouse / Apartment / Summerhouse), `purchase_price`, `%_change_between_offer_and_purchase`, `sqm`, `sqm_price`, `year_build`, `inflation`, `interest`, `yield`

# Steps Followed-:

Step 1: Loaded the `Housing Data csv` dataset into Power BI Desktop from the source file.

Step 2: Opened Power Query Editor and, under the **View** tab's Data Preview section, enabled **Column Distribution**, **Column Quality**, and **Column Profile** for all columns.

Step 3: Set column profiling to be based on the **entire dataset** (not the default 1000-row sample) to get an accurate read on nulls and data quality across all rows.

Step 4: Verified data types across all columns — `date` set to **Date**, `purchase_price` / `sqm` / `sqm_price` / `year_build` / `inflation` / `interest` / `yield` set to **Numeric**, `city` / `region` / `area` / `sales_type` / `house_type` set to **Text**.

Step 5: Checked all columns for errors and empty values using Column Quality/Profile. Found null values (<1% of records) in three columns: `city`, `annual inflation rate %`, and `yield on mortgage credit bond %`. No other errors or inconsistent rows were found — the rest of the dataset was clean.

Step 6: Resolved the nulls found in Step 5:
- `city` (Text)→ Replace Values → replaced nulls with "Unknown" (no meaningful numeric substitute exists for a text field)
- `annual inflation rate %` (Numeric) → checked **Column Profile / Value Distribution** to find the most frequently occurring value (**1.85**)→ Replace Values → replaced nulls with **1.85**
- `yield on mortgage credit bond %` (Numeric) → checked **Column Profile / Value Distribution** to find the most frequently occurring value (**1.47**) →Replace Values → replaced nulls with **1.47**

Nulls in numeric columns were replaced with the **mode (most frequent value)** rather than the mean/median, to preserve the natural distribution of the data.

Step 7: Built calculated columns for downstream analysis:
- `Age` — property age at time of sale (transaction year − `year_build`)
- `Offer Price` — back-calculated original offer price from `purchase_price` and `%_change_between_offer_and_purchase`

Step 8:In Report View, applied a consistent color theme and layout across all report pages for visual consistency.

Step 9: Wrote DAX measures for every KPI card and chart (full list in the section below), built the corresponding card/chart visuals, and added slicers for *Area*, *City*, and *Sales Type* to enable drill-down analysis across all pages.

Step 10:Analyzed the report views to identify patterns in regional price growth, offer-vs-purchase price alignment, sales-type performance, and property-type economics across the portfolio.

# DAX Measures & Calculated Columns

## Calculated Columns-
```dax
Age = YEAR('Housing Data csv'[date].[Date]) - 'Housing Data csv'[year_build]
```
Computes property age at time of sale (transaction year − year built).

```dax
Offer Price = (100 * 'Housing Data csv'[purchase_price]) / (100 - 'Housing Data csv'[%_change_between_offer_and_purchase])
```
Back-calculates the original listing/offer price from the purchase price and the percentage change between offer and purchase.
A card visual[Snap,Age,offer price]<img width="278" height="777" alt="Image" src="https://github.com/user-attachments/assets/40219602-0527-445a-a881-3aeee0fcf98f" />

## Measures-
```dax
Average Price SQM = AVERAGE('Housing Data csv'[sqm_price])
```
Average price per square meter across the current filter context.
A card visual was used to Average Price SQM- Use donut chart [Snap Average Price SQM]<img width="607" height="324" alt="Image" src="https://github.com/user-attachments/assets/d15733c5-af4e-4791-97be-19d8778e58e7" />


```dax
Last 12 Month Sales = 
CALCULATE(
    SUM('Housing Data csv'[purchase_price]),
    DATESINPERIOD('Housing Data csv'[date], MAX('Housing Data csv'[date]), -12, MONTH)
)
```
Rolling 12-month total sales ending on the latest date in context.
A card visual was used to Last 12 Month Sales- kpi card[Snap Last 12 Month Sales]<img width="284" height="142" alt="Image" src="https://github.com/user-attachments/assets/61446623-c3d0-4ba1-ad90-b84008a4d028" />

```dax
Median sales price change = 
VAR CurrMedianPrice =
    MEDIANX(
        FILTER('Housing Data csv', YEAR('Housing Data csv'[date].[Date]) = YEAR(MAX('Housing Data csv'[date].[Date]))),
        'Housing Data csv'[purchase_price]
    )
VAR PreMedianPrice =
    MEDIANX(
        FILTER('Housing Data csv', YEAR('Housing Data csv'[date].[Date]) = YEAR(MAX('Housing Data csv'[date].[Date])) - 1),
        'Housing Data csv'[purchase_price]
    )
RETURN
    IF(PreMedianPrice <> 0, (CurrMedianPrice - PreMedianPrice) / PreMedianPrice, BLANK())
```
Year-over-year percentage change in **median** purchase price (used in the region bar chart).
A card visual was used to Median sales price change- Use Bar chart [Snap Median sales price change]<img width="517" height="365" alt="Image" src="https://github.com/user-attachments/assets/47ddb5f0-ead2-4371-8031-d71b0e668f80" />

```dax
Offer to SQM Ratio = DIVIDE(SUM('Housing Data csv'[Offer Price]), SUM('Housing Data csv'[sqm]))
```
Offer price normalized per square meter, broken down by sales type.
A card visual was used to Offer to SQM Ratio- Use clustered bar chart [Snap Offer to SQM Ratio]<img width="446" height="330" alt="Image" src="https://github.com/user-attachments/assets/3c2e1f73-a998-4b8d-b8ba-3a15ef955556" />

```dax
Sales by Region = 
CALCULATE(SUM('Housing Data csv'[purchase_price]), ALLEXCEPT('Housing Data csv', 'Housing Data csv'[region]))
```
Total sales per region, ignoring all other active filters except region (keeps the funnel chart stable regardless of slicer context).
A card visual was used to Sales by Region- Use stacked bar chart [Snap Sales by Region]<img width="493" height="762" alt="Image" src="https://github.com/user-attachments/assets/9fe0ba3a-9127-44b2-b747-28f75955b7d8" />

```dax
TotalYTD Sales = TOTALYTD(SUM('Housing Data csv'[purchase_price]), 'Housing Data csv'[date].[Date])
```
Year-to-date cumulative sales.
A card visual was used to TotalYTD Sales- Use table chart [Snap TotalYTD Sales]<img width="646" height="410" alt="Image" src="https://github.com/user-attachments/assets/930e8841-e2e8-4ac1-8ba4-9b424d7ba05c" />

```dax
Unit Solds in latest Year & Quarter = 
CALCULATE(
    DISTINCTCOUNT('Housing Data csv'[house_id]),
    YEAR('Housing Data csv'[date]) = YEAR(MAX('Housing Data csv'[date])) &&
    QUARTER('Housing Data csv'[date]) = QUARTER(MAX('Housing Data csv'[date]))
)
```
Distinct count of houses sold in the most recent year-quarter combination.
A card visual was used to Unit Solds in latest Year & Quarter- Use card visual [Snap Unit Solds in latest Year & Quarter]<img width="267" height="158" alt="Image" src="https://github.com/user-attachments/assets/ddf4c7a4-92b6-4010-8830-3f838dbd959f" />


```dax
YOY Sales Growth = 
VAR CurrYearSales =
    CALCULATE(SUM('Housing Data csv'[purchase_price]), YEAR('Housing Data csv'[date]) = YEAR(MAX('Housing Data csv'[date])))
VAR Prevyearsales =
    CALCULATE(SUM('Housing Data csv'[purchase_price]), YEAR('Housing Data csv'[date]) = YEAR(MAX('Housing Data csv'[date])) - 1)
RETURN
    IF(Prevyearsales <> 0, (CurrYearSales - Prevyearsales) / Prevyearsales, BLANK())
```
Year-over-year percentage change in **total** sales value, sliced by sales type in the overview page.
A card visual was used to YOY Sales Growth- Use Line chart [Snap YOY Sales Growth]<img width="1573" height="334" alt="Image" src="https://github.com/user-attachments/assets/09d42038-e310-412f-9fe3-efc733823700" />

# Key Insights
1. **Zealand dominates the market**, contributing ~9.0bn in sales (~37.8% of total price/SQM share) — more than the next two regions combined.
2. **Regional price growth is uneven** — one region grew median price by +23% YoY while another declined by -6%, indicating a two-speed market.
3. **Offer price tracks purchase price almost perfectly** (near-linear scatter plot), suggesting minimal negotiation slippage across most transactions — outliers at the high end warrant a closer look.
4. **Family sales are shrinking sharply**, showing a ~-100% YoY decline, while auction, other, and regular sales stayed roughly flat — worth investigating whether this is a data/definition issue or a genuine market shift.
5. **Regular sales command the highest offer-to-SQM ratio (21K)**, more than 3x auction sales (6K) — auctions appear to close well below market value per square meter.
6. **Property age is a top driver of price**: newer homes (age 0–12) are associated with a **+1.78M** increase in average purchase price, per the AI Key Influencers visual.
7. **Farms are the largest but not the priciest per SQM** — despite having the biggest average size (196.6 sqm), Farms have the lowest total SQM-price contribution (5M), while Townhouses (107.7 sqm) contribute the most (58M), pointing to Townhouses being the most SQM-efficient/high-demand type.
8. **Villas yield the best return** among property types (3.4K yield), while Farms yield the least (0.3K) — relevant for investment-focused buyers.

DASHBOARD OVERVIEW -

[Snap of Housing market overview]<img width="1625" height="844" alt="Image" src="https://github.com/user-attachments/assets/e4f6215c-6d37-4dad-a701-4461f88aec9a" />

[Snap of Sales Performance]<img width="1666" height="865" alt="Image" src="https://github.com/user-attachments/assets/5349a057-cf6c-4537-9454-97e9fae81c10" />

[Snap of House type analysis]<img width="1620" height="844" alt="Image" src="https://github.com/user-attachments/assets/4c54bd26-c379-4a77-b2f1-6fbc2d120ac6" />

# Tools Used
- Power BI Desktop(data modeling, DAX, visualization)
- DAX for calculated columns and measures

#Files
- `Housing Data csv` — source transaction data
- Power BI `.pbix` report with 3 pages: Housing Market Overview, Sales Performance, Property Type Analysis

## 🚀 Future Improvements
- Add a dedicated **date/calendar table** for cleaner time intelligence (instead of relying on the raw date column)
- Investigate the **family_sale -100% YoY** anomaly for data quality issues
- Add **outlier detection** on the offer vs. purchase price scatter plot
- Incorporate **forecasting** (e.g., sales trend forecast) for the next 2–4 quarters
- Add **drill-through pages** from region/house-type summaries to transaction-level detail

How to Use -
Open the .pbix file in Power BI Desktop.
Use the slicers to filter.
