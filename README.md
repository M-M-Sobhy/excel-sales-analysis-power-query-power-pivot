# Excel Sales Analysis: Power Query, Power Pivot & Dashboard

> End-to-end Excel sales analytics: Power Query cleaning of messy CSVs, a Power Pivot star schema with DAX measures and a Calendar table, pivot tables, and an interactive dashboard for sales, orders and regional insights.

---

## Table of Contents
1. [Project Overview](#1-project-overview)
2. [Tools & Skills](#2-tools--skills)
3. [Dataset](#3-dataset)
4. [Stage 1: Data Cleaning with Power Query](#4-stage-1-data-cleaning-with-power-query)
5. [Stage 2: Data Modeling with Power Pivot](#5-stage-2-data-modeling-with-power-pivot)
6. [Stage 3: Pivot Tables](#6-stage-3-pivot-tables)
7. [Stage 4: Dashboard](#7-stage-4-dashboard)
8. [Key Insights](#8-key-insights)
9. [Power Query Source Code (M)](#9-power-query-source-code-m)
10. [Repository Structure](#10-repository-structure)
11. [How to Use](#11-how-to-use)

---

## 1. Project Overview
This project turns raw, inconsistent sales files into a clean analytical model and an interactive Excel dashboard. The workflow has four stages:

```
Raw CSV files  →  Power Query (clean & transform)  →  Power Pivot (star schema + DAX)  →  Pivot Tables  →  Dashboard
```

The raw data came as three sources: a **Customers** file, a **Products** file, and a **folder of monthly Orders files**. The data had mixed date formats, text-formatted prices, inconsistent casing and spacing, blank rows and errors. All of this was fixed in Power Query before modeling.

## 2. Tools & Skills
| Area | Tool / Feature |
|---|---|
| Data cleaning | Power Query (M language), folder combine, custom functions |
| Data modeling | Power Pivot, star schema, relationships, Calendar table, date hierarchy |
| Calculations | DAX measures |
| Analysis | PivotTables |
| Visualization | Excel dashboard |

## 3. Dataset
| Table | Type | Description |
|---|---|---|
| **Orders** | Fact | One row per order line: OrderID, CustomerID, ProductID, Date, Quantity (loaded from a folder of monthly files) |
| **Products** | Dimension | ProductID, ProductName, Category, Price |
| **Customers** | Dimension | CustomerID, customer name (First Name / Second Name), Region |
| **Calendar** | Dimension | Date, Year, Month Number, Month, MMM-YYYY, Day Of Week Number, Day Of Week |

Scope: **19,586 orders**, **108,087 units sold**, **25 products** in **5 categories**, customers across **5 regions** (Alexandria, Aswan, Cairo, Giza, Mansoura), year **2024**.

---

## 4. Stage 1: Data Cleaning with Power Query

### 4.1 Customers
| Step | Purpose |
|---|---|
| Promote headers, set data types | Use the first row as column names and fix types |
| Convert `CustomerID` to **Text** | The ID is a key, not a number to aggregate |
| Trim `CustomerName` | Remove leading and trailing spaces |
| Capitalize Each Word on name and region | Consistent casing (`Text.Proper`) |
| Replace `" "` with `""` in `Region`, then capitalize again | Fix region values that contained stray spaces |

### 4.2 Products
| Step | Purpose |
|---|---|
| Promote headers, set data types | Read `Price` as text first so dirty values do not break the load |
| Convert `ProductID` to **Text** | Key column |
| Replace errors in `ProductName` with `"Not Detect"` | Keep problem rows visible instead of dropping them |
| Trim and capitalize `ProductName` and `Category` | Consistent text values |
| Remove `$` from `Price` | Strip currency symbol |
| Replace `"Two Hundred"` with `200` | Fix a price typed as words |
| Change `Price` to **Number** | Enable calculations |

### 4.3 Orders
The Orders query is the most complex because the data arrives as multiple monthly files with inconsistent date formats.

| Step | Purpose |
|---|---|
| Load from **Folder**, filter hidden files | Combine all monthly Orders files into one table via a custom function |
| Keep `Source.Name` + expanded content | Keep the file name, which contains the month number |
| Remove blank rows and rows with null `ProductID` / `Quantity` | Drop empty or incomplete records |
| Convert `OrderID`, `CustomerID`, `ProductID` to **Text** | Key columns |
| Extract **Month** from the file name | Split by character transition (letters vs. digits) |
| Normalize `/` to `-` and split `OrderDate` into 3 parts | Handle mixed formats such as `DD/MM/YYYY`, `YYYY-MM-DD` and others |
| Detect **Year** (part > 2000) and **Day** (part ≤ 31 and ≠ Month) | Rebuild the date correctly whatever the original order of parts |
| Merge Month / Day / Year into `New Date` and set type **Date** | One clean date column |
| Merge with **Products** (Left Outer) | Bring in `ProductName` and `Price` |
| Add **Total Sales** = `Price × Quantity` | Revenue per order line |
| Merge with **Customers** (Left Outer) | Bring in `CustomerName` and `Region` |

---

## 5. Stage 2: Data Modeling with Power Pivot

The cleaned tables were loaded into the Excel Data Model and arranged as a **star schema**: one fact table (**Orders**) surrounded by three dimension tables.

<img width="1175" height="624" alt="Screenshot 2026-11-14 222532" src="https://github.com/user-attachments/assets/19fafec1-6cb3-4306-b122-0d7f04c0cd0d" />


### Relationships
| From (One) | To (Many) | Key |
|---|---|---|
| Products | Orders | `ProductID` |
| Customers | Orders | `CustomerID` |
| Calendar | Orders | `Date` |

### Calendar table
A dedicated Calendar table supports time intelligence. It contains `Date`, `Year`, `Month Number`, `Month`, `MMM-YYYY`, `Day Of Week Number`, `Day Of Week`, and a **Date Hierarchy** (Year → Month → Date).

### DAX measures (in Orders)
| Measure | Meaning |
|---|---|
| **Total Sales** | Total revenue |
| **Total Quantity** | Total units sold |
| **Total Orders** | Number of orders |
| **Total Customers** | Number of distinct customers |

---

## 6. Stage 3: Pivot Tables
PivotTables were built on the Data Model and use the DAX measures above.

<img width="744" height="462" alt="image" src="https://github.com/user-attachments/assets/7e0a9b20-a054-41a5-80b0-76069dd2c01b" />


| Pivot Table | Rows | Values |
|---|---|---|
| Sales by Category | Category | Total Sales |
| Quantity by Product | Product Name (by Year) | Total Quantity |
| Orders by Region | Region | Total Orders |
| Monthly Performance | Month | Total Sales, Total Orders |

---

## 7. Stage 4: Dashboard
The pivot tables feed the final dashboard, which brings together the main KPIs and charts on a single page.

<img width="1010" height="568" alt="Screenshot 2026-08-25 225447" src="https://github.com/user-attachments/assets/5bf7adde-7655-494a-9941-c70192446866" />


---

## 8. Key Insights
- **Clothing** is the top category by sales (about $29.1M), followed by **Toys** (about $25.4M). **Furniture** is the lowest (about $10.4M).
- Total sales across the five categories are roughly **$101.1M**.
- **Mansoura** and **Aswan** place the most orders (4,046 and 4,001); **Alexandria** places the fewest (3,654).
- Monthly orders are very stable (about 1,623 to 1,641 per month). **May** has the highest sales (about $8.66M) and **December** the lowest (about $8.29M).
- **Tablet** and **Table** lead in units sold (4,634 and 4,620); **Phone** sells the fewest (3,894).

---

## 9. Power Query Source Code (M)

> Replace the file paths with your own folder locations.

<details>
<summary><b>Customers query</b></summary>

```m
let
    Source = Csv.Document(File.Contents("<path>\Customers.csv"),[Delimiter=",", Columns=3, Encoding=1252, QuoteStyle=QuoteStyle.None]),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"CustomerID", Int64.Type}, {"CustomerName", type text}, {"Region", type text}}),
    #"Changed Type1" = Table.TransformColumnTypes(#"Changed Type",{{"CustomerID", type text}}),
    #"Trimmed Text" = Table.TransformColumns(#"Changed Type1",{{"CustomerName", Text.Trim, type text}}),
    #"Capitalized Each Word" = Table.TransformColumns(#"Trimmed Text",{{"CustomerName", Text.Proper, type text}, {"Region", Text.Proper, type text}}),
    #"Replaced Value" = Table.ReplaceValue(#"Capitalized Each Word"," ","",Replacer.ReplaceText,{"Region"}),
    #"Capitalized Each Word1" = Table.TransformColumns(#"Replaced Value",{{"Region", Text.Proper, type text}}),
    #"Filtered Rows" = Table.SelectRows(#"Capitalized Each Word1", each true)
in
    #"Filtered Rows"
```
</details>

<details>
<summary><b>Products query</b></summary>

```m
let
    Source = Csv.Document(File.Contents("<path>\Products.csv"),[Delimiter=",", Columns=4, Encoding=1252, QuoteStyle=QuoteStyle.None]),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"ProductID", Int64.Type}, {"ProductName", type text}, {"Category", type text}, {"Price", type text}}),
    #"Changed Type1" = Table.TransformColumnTypes(#"Changed Type",{{"ProductID", type text}}),
    #"Replaced Errors" = Table.ReplaceErrorValues(#"Changed Type1", {{"ProductName", "Not Detect"}}),
    #"Trimmed Text" = Table.TransformColumns(#"Replaced Errors",{{"ProductName", Text.Trim, type text}}),
    #"Capitalized Each Word" = Table.TransformColumns(#"Trimmed Text",{{"ProductName", Text.Proper, type text}}),
    #"Trimmed Text1" = Table.TransformColumns(#"Capitalized Each Word",{{"Category", Text.Trim, type text}}),
    #"Capitalized Each Word1" = Table.TransformColumns(#"Trimmed Text1",{{"Category", Text.Proper, type text}}),
    #"Replaced Value" = Table.ReplaceValue(#"Capitalized Each Word1","$","",Replacer.ReplaceText,{"Price"}),
    #"Replaced Value1" = Table.ReplaceValue(#"Replaced Value","Two Hundred","200",Replacer.ReplaceText,{"Price"}),
    #"Changed Type2" = Table.TransformColumnTypes(#"Replaced Value1",{{"Price", type number}})
in
    #"Changed Type2"
```
</details>

<details>
<summary><b>Orders query</b></summary>

```m
let
    Source = Folder.Files("<path>\Orders"),
    #"Filtered Hidden Files1" = Table.SelectRows(Source, each [Attributes]?[Hidden]? <> true),
    #"Invoke Custom Function1" = Table.AddColumn(#"Filtered Hidden Files1", "Transform File", each #"Transform File"([Content])),
    #"Renamed Columns1" = Table.RenameColumns(#"Invoke Custom Function1", {"Name", "Source.Name"}),
    #"Removed Other Columns1" = Table.SelectColumns(#"Renamed Columns1", {"Source.Name", "Transform File"}),
    #"Expanded Table Column1" = Table.ExpandTableColumn(#"Removed Other Columns1", "Transform File", Table.ColumnNames(#"Transform File"(#"Sample File"))),
    #"Changed Type" = Table.TransformColumnTypes(#"Expanded Table Column1",{{"Source.Name", type text}, {"OrderID", Int64.Type}, {"CustomerID", Int64.Type}, {"ProductID", Int64.Type}, {"OrderDate", type text}, {"Quantity", Int64.Type}}),
    #"Removed Blank Rows" = Table.SelectRows(#"Changed Type", each not List.IsEmpty(List.RemoveMatchingItems(Record.FieldValues(_), {"", null}))),
    #"Filtered Rows" = Table.SelectRows(#"Removed Blank Rows", each ([ProductID] <> null)),
    #"Changed Type1" = Table.TransformColumnTypes(#"Filtered Rows",{{"OrderID", type text}, {"CustomerID", type text}, {"ProductID", type text}}),
    #"Filtered Rows1" = Table.SelectRows(#"Changed Type1", each ([Quantity] <> null)),
    #"Duplicated Column" = Table.DuplicateColumn(#"Filtered Rows1", "OrderDate", "OrderDate - Copy"),
    #"Split Column by Character Transition" = Table.SplitColumn(#"Duplicated Column", "Source.Name", Splitter.SplitTextByCharacterTransition((c) => not List.Contains({"0".."9"}, c), {"0".."9"}), {"Source.Name.1", "Source.Name.2"}),
    #"Removed Columns" = Table.RemoveColumns(#"Split Column by Character Transition",{"Source.Name.1"}),
    #"Split Column by Character Transition1" = Table.SplitColumn(#"Removed Columns", "Source.Name.2", Splitter.SplitTextByCharacterTransition({"0".."9"}, (c) => not List.Contains({"0".."9"}, c)), {"Source.Name.2.1", "Source.Name.2.2"}),
    #"Removed Columns1" = Table.RemoveColumns(#"Split Column by Character Transition1",{"Source.Name.2.2"}),
    #"Renamed Columns" = Table.RenameColumns(#"Removed Columns1",{{"Source.Name.2.1", "Month"}}),
    #"Replaced Value" = Table.ReplaceValue(#"Renamed Columns","/","-",Replacer.ReplaceText,{"OrderDate - Copy"}),
    #"Split Column by Delimiter" = Table.SplitColumn(#"Replaced Value", "OrderDate - Copy", Splitter.SplitTextByDelimiter("-", QuoteStyle.Csv), {"OrderDate - Copy.1", "OrderDate - Copy.2", "OrderDate - Copy.3"}),
    #"Changed Type2" = Table.TransformColumnTypes(#"Split Column by Delimiter",{{"Month", Int64.Type}, {"OrderDate - Copy.1", Int64.Type}, {"OrderDate - Copy.2", Int64.Type}, {"OrderDate - Copy.3", Int64.Type}}),
    #"Added Conditional Column" = Table.AddColumn(#"Changed Type2", "Year", each if [#"OrderDate - Copy.1"] > 2000 then [#"OrderDate - Copy.1"] else if [#"OrderDate - Copy.2"] > 2000 then [#"OrderDate - Copy.2"] else if [#"OrderDate - Copy.3"] > 2000 then [#"OrderDate - Copy.3"] else "Not Found"),
    #"Reordered Columns" = Table.ReorderColumns(#"Added Conditional Column",{"OrderID", "CustomerID", "ProductID", "OrderDate", "Quantity", "OrderDate - Copy.1", "OrderDate - Copy.2", "OrderDate - Copy.3", "Month", "Year"}),
    #"Added Custom" = Table.AddColumn(#"Reordered Columns", "Day", each if [#"OrderDate - Copy.1"] <= 31 and [#"OrderDate - Copy.1"] <> [Month] then [#"OrderDate - Copy.1"]
else if [#"OrderDate - Copy.2"] <= 31 and [#"OrderDate - Copy.2"] <> [Month] then [#"OrderDate - Copy.2"]
else if [#"OrderDate - Copy.3"] <= 31 and [#"OrderDate - Copy.3"] <> [Month] then [#"OrderDate - Copy.3"]
else [Month]),
    #"Reordered Columns1" = Table.ReorderColumns(#"Added Custom",{"OrderID", "CustomerID", "ProductID", "OrderDate", "Quantity", "OrderDate - Copy.1", "OrderDate - Copy.2", "OrderDate - Copy.3", "Month", "Year", "Day"}),
    #"Removed Columns2" = Table.RemoveColumns(#"Reordered Columns1",{"OrderDate - Copy.1", "OrderDate - Copy.2", "OrderDate - Copy.3"}),
    #"Changed Type3" = Table.TransformColumnTypes(#"Removed Columns2",{{"Month", Int64.Type}, {"Year", Int64.Type}, {"Day", Int64.Type}}),
    #"Merged Columns" = Table.CombineColumns(Table.TransformColumnTypes(#"Changed Type3", {{"Month", type text}, {"Day", type text}, {"Year", type text}}, "en-US"),{"Month", "Day", "Year"},Combiner.CombineTextByDelimiter("/", QuoteStyle.None),"New Date"),
    #"Changed Type4" = Table.TransformColumnTypes(#"Merged Columns",{{"New Date", type date}, {"OrderDate", type date}}),
    #"Removed Columns3" = Table.RemoveColumns(#"Changed Type4",{"OrderDate"}),
    #"Reordered Columns2" = Table.ReorderColumns(#"Removed Columns3",{"OrderID", "CustomerID", "ProductID", "New Date", "Quantity"}),
    #"Merged Queries" = Table.NestedJoin(#"Reordered Columns2", {"ProductID"}, Products, {"ProductID"}, "Products", JoinKind.LeftOuter),
    #"Expanded Products" = Table.ExpandTableColumn(#"Merged Queries", "Products", {"ProductName", "Price"}, {"ProductName", "Price"}),
    #"Added Custom1" = Table.AddColumn(#"Expanded Products", "Total Sales", each [Price]*[Quantity]),
    #"Merged Queries1" = Table.NestedJoin(#"Added Custom1", {"CustomerID"}, Customers, {"CustomerID"}, "Customers", JoinKind.LeftOuter),
    #"Expanded Customers" = Table.ExpandTableColumn(#"Merged Queries1", "Customers", {"CustomerName", "Region"}, {"CustomerName", "Region"}),
    #"Filtered Rows2" = Table.SelectRows(#"Expanded Customers", each true)
in
    #"Filtered Rows2"
```
</details>

---

## 10. Repository Structure
```
├── data/
│   ├── Customers.csv
│   ├── Products.csv
│   └── Orders/                  # monthly order files
├── power-query/
│   ├── Customers.pq
│   ├── Products.pq
│   └── Orders.pq
├── assets/
│   ├── star_schema.png
│   ├── pivot_tables.png
│   └── dashboard.png
├── Sales_Analysis.xlsx          # data model, pivot tables, dashboard
└── README.md
```

## 11. How to Use
1. Download the repository and open `Sales_Analysis.xlsx` in Excel (Microsoft 365 or Excel 2016+ with Power Pivot enabled).
2. In Power Query, update the source paths for `Customers.csv`, `Products.csv` and the `Orders` folder.
3. Click **Data → Refresh All** to reload the data, the model and the pivot tables.
4. Open the **Dashboard** sheet to explore the results.

---

⭐ If you found this project useful, feel free to star the repository.
