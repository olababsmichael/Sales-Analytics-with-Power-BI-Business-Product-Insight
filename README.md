# 📊 Sales Data Analysis Project

![Power BI](https://img.shields.io/badge/Tool-PowerBI-F2C811?logo=powerbi)
![DAX](https://img.shields.io/badge/Language-DAX-blue)
![Dataset](https://img.shields.io/badge/Dataset-Sales%20Analytics-green)
![Status](https://img.shields.io/badge/Project-Completed-brightgreen)
 
# 🔖 Project Overview

This project focuses on analyzing a multi-country sales dataset to uncover actionable insights for business decision-making. Using **Power BI**, **Power Query**, and **DAX**, the analysis examines revenue trends, product performance, market segmentation, and geographic contributions.

The main objectives are to:

- Understand overall **sales performance** and growth trends.
- Identify **top-performing products and manufacturers**.
- Analyze **urban vs rural revenue contribution**.
- Compare **country-wise revenue** and segment performance.
- Provide a clear **dashboard visualization** for stakeholders to monitor KPIs and make informed decisions.

The project delivers **two interactive dashboards**:

1. **Business Overview Dashboard** — High-level KPIs, revenue trends, and urban/rural distribution.
2. **Market & Product Insights Dashboard** — Detailed analysis of product categories, segments, manufacturers, and country-level performance.

This project demonstrates **end-to-end data analysis**, from cleaning and modeling to visualization and insight generation, enabling data-driven strategies for revenue optimization and market planning.

# 📁 Data Preparation & Cleaning

### ✔ Steps Performed  
- Imported datasets into Power Query.  
- Cleaned Sales Data: fixed spelling errors, standardized country names, ensured numeric column integrity.  
- Cleaned Manufacturer table: removed empty rows, transposed, removed redundant columns.  
- Combined Geo tables and standardized location fields.  
- Ensured all Product and Manufacturer IDs matched across tables.  

## ✅ Data Quality Checks

These checks were performed after ETL and before modelling:

- **Row counts & schema validation** — verified expected row counts after append and column names/types match spec.
- **Null value audit** — flagged columns with >5% nulls for review (Date, ProductID, Zip).
- **Outlier detection (Revenue & Units)** — used interquartile range and manual inspection to find implausible values.
- **Duplicate detection** — deduplicated transactional rows based on ProductID+Date+Zip+Revenue.
- **Referential integrity** — checked that every ProductID in Sales Data has a matching Product dimension entry; same for ManufacturerID and Zip.
- **Date continuity** — verified no large gaps in calendar dates for the reporting period.
- **Currency & Unit consistency** — confirmed revenue values are in a single currency (or documented conversion).

## 🔍 Validation & Testing

Validation steps carried out to ensure dashboard correctness:

- **Measure validation** — cross-validated Total Revenue and Top N results against Excel pivot tables.
- **Slicer & filter tests** — confirmed each slicer (Year, Country, Category) filters visuals and measures correctly.
- **Edge-case tests** — empty selection, multiple year selections, no-data slices tested to ensure safe DAX returns (e.g., `IF(ISBLANK(),0)`).
- **Performance testing** — tested refresh times and visual load times in Power BI Desktop (aim < 5s for main page).
- **Peer review** — another analyst reviewed model relationships, measure logic, and report layout.

# 📊 Data Modelling

This project uses a **Star Schema** with one Fact table and three Dimension tables.


## ⭐  Data Model Diagram
```
            ┌──────────────────┐
            │   Manufacturer   │
            │   (Dimension)    │
            └───────┬──────────┘
                    │ 1:*  
            ┌──────────────────┐
            │     Product      │
            │   (Dimension)    │
            └───────┬──────────┘
                    │ 1:*
            ┌──────────────────┐       ┌────────────────┐
            │    Sales Data    │-------│      Geo       │
            │      (Fact)      │ *:*   │  (Dimension)   │
            └──────────────────┘       └────────────────┘
```

# 📁  Tables & Columns

<details>
<summary>🧮 Fact Table: Sales Data</summary>

| Column | Description |
|--------|-------------|
| ProductID | Link to Product |
| Date | Sales date |
| Zip | Link to Geo |
| Units | Quantity sold |
| Revenue | Total revenue |
| Country | Country of sale |

</details>

<details>
<summary>🗂 Dimension Table: Product</summary>

| Column | Description |
|--------|-------------|
| ProductID | Primary key |
| Product | Product name |
| Category | Product category |
| ManufacturerID | Link to Manufacturer |
| Price | Unit price |

</details>

<details>
<summary>🧰 Dimension Table: Manufacturer</summary>

| Column | Description |
|--------|-------------|
| ManufacturerID | Primary key |
| Manufacturer | Manufacturer name |

</details>

<details>
<summary>🌍 Dimension Table: Geo</summary>

| Column | Description |
|--------|-------------|
| Zip | Primary key |
| City | City |
| State | State |
| Region | Region |
| District | District |
| Country | Country |

</details>

## 🔗  Relationships

| From Table | Column | To Table | Column | Type |
|------------|--------|----------|--------|------|
| Sales Data | ProductID | Product | ProductID | Many-to-One |
| Product | ManufacturerID | Manufacturer | ManufacturerID | Many-to-One |
| Sales Data | Zip | Geo | Zip | Many-to-Many |

✔ Cross-filter direction: *Single*  
✔ Referential integrity enabled where possible  


## 📐  DAX Measures

<details> <summary>💰 Total Revenue & Units</summary>
            
### **Total Revenue**
```DAX
Total Revenue = SUM('Sales Data'[Revenue])
```

### **Total Units Sold**
```DAX
Total Units = SUM('Sales Data'[Units])
```
</details> <details> <summary>📦 Counts of Products, Manufacturers, Countries & Cities</summary>
            
### **Total Products**
```DAX
Total Products = DISTINCTCOUNT('Product'[ProductID])
```

### **Total Manufacturers**
```DAX
Total Manufacturers = DISTINCTCOUNT('Manufacturer'[ManufacturerID])
```

### **Total Countries**
```DAX
Total Countries = DISTINCTCOUNT('Geo'[Country])
```

### **Total Cities**
```DAX
Total Cities = DISTINCTCOUNT('Geo'[City])
```
</details> <details> <summary>🏙 Urban Revenue Metrics</summary>
            
### **Urban Revenue**
```DAX
Urban Revenue = CALCULATE([Total Revenue], 'Geo'[Region] = "Urban")
```

### **% Urban Revenue**
```DAX
% Urban Revenue =
DIVIDE(
    [Urban Revenue],
    [Total Revenue]
)
```
</details> <details> <summary>📊 Revenue by Category & Top Performers</summary>
            
### **Revenue by Category**
```DAX
Revenue by Category = SUMX(VALUES(Product[Category]), [Total Revenue])
```

### **Top 5 Manufacturers by Revenue**
```DAX
Top Manufacturers Revenue =
TOPN(5, SUMMARIZE(Manufacturer, Manufacturer[Manufacturer], "Revenue", [Total Revenue]), [Total Revenue], DESC)
```

### **Top 10 Products by Revenue**
```DAX
Top Products Revenue =
TOPN(10, SUMMARIZE(Product, Product[Product], "Revenue", [Total Revenue]), [Total Revenue], DESC)
```
</details> <details> <summary>📈 Revenue Trend</summary>
            
### **Revenue Trend (Year-Month)**
```DAX
Revenue Trend = CALCULATE([Total Revenue], ALLEXCEPT('Sales Data', 'Date'[Year], 'Date'[Month]))
```
</details>

## 📋 Metrics Dictionary

This table defines the KPIs used across the dashboard and recommended display formats.

| Metric | Definition / DAX (summary) | Format / Card |
|---|---:|:---|
| Total Revenue | Sum of revenue for the selected filter context. `Total Revenue = SUM('Sales Data'[Revenue])` | Currency (e.g., $#,##0.00) |
| Total Units | Sum of units sold. `Total Units = SUM('Sales Data'[Units])` | Integer |
| Total Products | Unique count of ProductID. `DISTINCTCOUNT('Product'[ProductID])` | Integer |
| Total Manufacturers | Unique count of ManufacturerID. `DISTINCTCOUNT('Manufacturer'[ManufacturerID])` | Integer |
| Total Countries | Unique count of Country values. `DISTINCTCOUNT('Geo'[Country])` | Integer |
| % Urban Revenue | Urban revenue divided by total revenue. `DIVIDE([Urban Revenue],[Total Revenue])` | Percent (0.00%) |
| YoY Revenue | Year-over-year change in total revenue using `DATEADD()` | Percent with arrow indicator |
| Top 10 Products | Top N list using `TOPN()` on revenue | Table / Bar Visual |
| Top 5 Manufacturers | Top N list using `TOPN()` on revenue | Table / Bar Visual |

# 📊 Dashboard Section

##  Dashboard 1 — Business Overview
This dashboard provides a high-level overview of sales performance, revenue, and urban vs rural contribution across all regions.

![Dashboard 1](/salesdata/dashboard1.JPG)
### **Insights**
- Urban regions contribute the majority of revenue.
- Revenue is concentrated among a few top manufacturers.
- Growth trends are visible year-over-year.

##  Dashboard 2 — Market & Product Insights
This dashboard focuses on deeper insights into products, segments, countries, and market performance.

![Dashboard 2](/salesdata/dashboard2.JPG)
### **Insights**
- Convenience and Moderation segments lead revenue.
- Some segments (Youth, Regular) require targeted interventions.
- Countries like USA lead sales; others show growth opportunities.

# 📌 Additional Insights

- Highest selling products by category  
- Revenue performance across regions  
- Seasonal sales trends based on monthly data  

## ▶️  Run Locally

Requirements:
- Power BI Desktop (latest stable release)  
- Local copies of the CSV files in `/data` folder

Steps:
1. Clone or download the repository.  
2. Open `My Sales Data.pbix` in Power BI Desktop.  
3. If you want to re-run ETL: open **Transform Data** → check Applied Steps for each query and refresh data sources from `/data`.  
4. To refresh visuals after replacing CSVs, click **Refresh** (Home → Refresh).  
5. If Date table needs extension, open the Date table query or use the DAX calendar code in Appendix.

Notes:
- If data file paths change, update source paths in Power Query (Home → Data source settings).  
- Do not change field names used in measures unless you update DAX accordingly.
---

## ✅ Conclusion  

This project demonstrates strong analytical workflow: cleaning, modelling, DAX, and visualization. It highlights the ability to transform raw data into actionable insights.

---

## Project Files Structure
- **data**
  - [Dimension Tables(product, manufacturer, geo)](./salesdata/bi_dimensions.xlsx)

- **images**
  - [dashboard1](./salesdata/dashboard1.JPG)
  - [dashboard2](./salesdata/dashboard2.JPG)
  - [model_diagram](./salesdata/relationship.png)

## 📜 License & Attribution

This project is released under the **MIT License** — feel free to reuse and adapt.  
If you republish parts of this project, please attribute the author.

## ✉️ Contact

**Author:** Olaoluwa Michael Babawale  
**Email:** olababstheanlyst@gmail.com  

