# Zepto Inventory Analysis (PostgreSQL)

SQL analysis of a Zepto grocery catalog covering pricing, discounts, stock availability and inventory.

## Overview

- **Dataset:** `zepto_v2.csv` with 7,462 product listings across 14 categories
- **Tools:** PostgreSQL 18, pgAdmin 4
- **SQL concepts used:** aggregate functions, `GROUP BY` / `HAVING`, `CASE WHEN`, `DISTINCT`, `ROUND`, sorting and `LIMIT`, `UPDATE` / `DELETE` for cleaning

## Dataset

| Column | Description |
|---|---|
| `sku_id` | Auto-generated primary key (not in the source file) |
| `category` | Product category (14 in total) |
| `name` | Product name |
| `mrp` | Maximum retail price |
| `discountPercent` | Discount offered on MRP (%) |
| `availableQuantity` | Units currently available |
| `discountedSellingPrice` | Price after discount |
| `weightInGms` | Product weight in grams |
| `outOfStock` | `TRUE` if the product is out of stock |
| `quantity` | Present in the source file; not documented, so not used in the analysis |

> Prices are stored in **paise** in the source file and converted to rupees during cleaning.

## Project structure

```
.
├── Zepto_SQL_data_analysis.sql   # schema, exploration, cleaning and analysis queries
├── zepto_v2.csv                  # dataset (remove this line if you don't upload it)
└── README.md
```

## How to run

1. Create a PostgreSQL database (for example `zept_SQL_project`).
2. Run the **Schema Creation** section of the SQL file.
3. Import `zepto_v2.csv` into the `zepto` table (leave out `sku_id`, it is generated automatically).
4. Run the **Data Exploration** section.
5. Run the **Data Cleaning** section **once only**.
6. Run the **Data Analysis** queries.

> **Warning:** do not re-run the schema section after importing, because `DROP TABLE` deletes your data. Do not re-run the paise-to-rupee `UPDATE` either, or prices will be divided by 100 again.

## Data exploration

- 7,462 rows and 14 categories
- No null values in any column
- 6,556 products in stock and 906 out of stock (about 12%)
- 1,680 product names appear more than once, so many products are listed in several rows

## Data cleaning

- Checked for products with `mrp = 0` or `discountedSellingPrice = 0`: none were found, so the `DELETE` removed no rows
- Converted `mrp` and `discountedSellingPrice` from paise to rupees (divided by 100)

## Business questions and key findings

| # | Question | Result |
|---|---|---|
| Q1 | Top 10 products by discount percentage | Dukes Waffy wafers lead at 51%; the rest of the top 10 are at 50% |
| Q2 | High-MRP (above ₹300) products that are out of stock | 4 products: Patanjali Cow's Ghee (₹565), MamyPoko Pants Extra Large (₹399), Aashirvaad Atta With Multigrains (₹315), Everest Kashmiri Lal Chilli Powder (₹310) |
| Q3 | Estimated revenue per category (selling price × available quantity) | Cooking Essentials and Munchies are tied at the top with ₹674,738 each; Fruits & Vegetables is lowest at ₹21,692 |
| Q4 | Products with MRP above ₹500 and discount below 10% | Mostly cooking oils in jars, many with 0 to 1% discount (for example Dhara Kachi Ghani Mustard Oil at 8%, Saffola Gold at 0%) |
| Q5 | Top 5 categories by average discount | Fruits & Vegetables 15.46%, Meats, Fish & Eggs 11.03%, then Packaged Food, Ice Cream & Desserts and Chocolates & Candies tied at 8.32% |
| Q6 | Price per gram for products of 100 g or more | Cheapest is Vicks Cough Drops Menthol at ₹0.0172/g, followed by iodised salt and onions at about ₹0.019/g (1,395 rows) |
| Q7 | Weight bands: Low (under 1 kg), Medium (1 to under 5 kg), Bulk (5 kg and above) | 1,782 distinct product and weight combinations classified |
| Q8 | Total inventory weight per category (grams) | Munchies and Cooking Essentials tied at 2,809,308 g; Meats, Fish & Eggs lowest at 96,032 g |

Q3, Q5 and Q8 are category-level results and are affected by the data quality issue below.

## Data quality note

Several categories return **identical totals**, which suggests the same products are listed under more than one category in the source data:

- Munchies and Cooking Essentials (revenue ₹674,738 and weight 2,809,308 g each)
- Personal Care and Paan Corner (revenue ₹541,698; weight 696,374 g)
- Packaged Food, Ice Cream & Desserts and Chocolates & Candies (revenue ₹448,770 for all three; weight 981,594 g for all three)
- Beverages and Dairy, Bread & Batter (revenue ₹110,102; weight 287,470 g)

Category rankings should therefore be read with caution. Product-level questions (Q1, Q2, Q4, Q6) are less affected because they use `DISTINCT` on product names.

## Limitations

- Revenue and weight are estimates based on **current stock**, not actual sales
- The `quantity` column is undocumented, so it is not used
- The source file has no product ID, so `sku_id` was generated on import
- Duplicate product names mean `DISTINCT` queries may still show the same product at different pack sizes

## Skills demonstrated

Data exploration, data cleaning, aggregation, conditional logic with `CASE WHEN`, and turning query output into business insights using PostgreSQL.
