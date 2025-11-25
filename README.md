# 💳 Transaction Analytics & Operational Insights | FinTech | Python
<img width="1658" height="922" alt="image" src="https://github.com/user-attachments/assets/3e31002c-e1d0-4b65-b781-c437190f4d9f" />

**Author:** Huong Le  
**Domain:** Digital Payments & E-commerce Analytics  
**Tools Used:** Python (Pandas, NumPy, Matplotlib, Seaborn)  

---

## 📌 Background & Overview

**Objective:**

**📖 What is this project about?**

This project aims to analyze **user behavior**, **transaction journeys**, and **product-team performance** within an **E-wallet platform**.

The goal is to:
- Identify top-performing and underperforming products
- Understand team operational performance
- Detect refund anomalies and high-risk contributors
- Categorize complex transaction patterns
- Reveal how users engage across different transaction types

**👤 Who is this project for?**
- **Business Analysts & Data Analysts**: understand user behavior, product performance, and transaction dynamics in an e-wallet ecosystem.
- **Fintech Product Manager**: need data insights about customer engagement, payment/transfer patterns, refund contributors, and product/team performance.
- **Engineering & Operations Teams**: rely on structured insights to improve transaction workflows, detect anomalies, and enhance customer experience.
  
---

## 📂 Dataset

### 📌 Data Source
- **Source**: Internal company database (e-wallet transaction records)  
- **Size**:  
  - `product.csv`: 493 rows × 3 columns  
  - `payment_report.csv`: 920 rows × 5 columns  
  - `transactions.csv`: 1,324,002 rows × 9 columns  
- **Format**: `.csv`  

---

### 📊 Data Structure 
This project utilizes **three** datasets:

1. **product.csv** – Product details, including category and team ownership.  
2. **payment_report.csv** – Monthly payment volume and source tracking.  
3. **transactions.csv** – Comprehensive transaction records.  

**Key Relationships:**
- `product_id` links **product.csv** and **payment_report.csv**.  
- `transaction_id` is the unique key in **transactions.csv** for tracking payments.  
- `source_id` in **payment_report.csv** contributes to refund analysis.  

<details>
  <summary><strong>Table 1: Products</strong></summary>

| Column Name  | Data Type | Description |
|-------------|----------|-------------|
| product_id  | INT      | Unique identifier for each product |
| category    | TEXT     | Product category |
| team_own    | TEXT     | Team responsible for the product |
</details>
<details>
  <summary><strong>Table 2: Payment Report</strong></summary>

| Column Name    | Data Type | Description |
|---------------|----------|-------------|
| report_month  | DATE     | Month of the payment report |
| payment_group | TEXT     | Type of payment (e.g., refund, purchase) |
| product_id    | INT      | Associated product ID |
| source_id     | INT      | Source of the transaction |
| volume        | FLOAT    | Total payment volume |
</details>
<details>
  <summary><strong>Table 3: Transactions</strong></summary>

| Column Name    | Data Type | Description |
|---------------|----------|-------------|
| transaction_id | INT      | Unique transaction identifier |
| merchant_id    | INT      | Merchant involved in the transaction |
| volume         | FLOAT    | Transaction amount |
| transType      | INT      | Type of transaction |
| transStatus    | TEXT     | Status of the transaction (e.g., completed, failed) |
| sender_id      | INT      | Sender of the transaction |
| receiver_id    | INT      | Receiver of the transaction |
| extra_info     | TEXT     | Additional details about the transaction |
| timeStamp      | TIMESTAMP | Time when the transaction occurred |
</details>

---
### ⚒️ Data Cleaning & Preprocessing 
- Loaded the three CSV files (`payment`, `product`, `transaction`) into Pandas DataFrames.  
- Checked basic structure: number of rows/columns, data types, and missing values.  
- Removed exact duplicate rows and dropped records missing **critical keys** (e.g., `product_id`, `volume`, `transType`).  
- Converted `report_month` to datetime for time-based filtering.  
- Merged `payment` and `product` tables into a single `payment_product` dataset using a **left join** on `product_id`.
  
<details> <summary><strong>Import Packages & Mount Google Drive</strong></summary>

```python
import pandas as pd
import numpy as np

from google.colab import drive
drive.mount('/content/drive')

path = '/content/drive/MyDrive/American Preparation/DAC K33/Python_Project 2/'
```
- Loads core Python libraries
- Mounts Google Drive to access raw CSV files
</details>
<details> <summary><strong>Load Raw Data Files</strong></summary>
  
```python  
df_payment = pd.read_csv(path + 'payment_report.csv')
df_product = pd.read_csv(path + 'product.csv')
df_transactions = pd.read_csv(path + 'transactions.csv')
```
- Keeps each dataset separate for flexible cleaning & transformation
- Ensures schema consistency before merging
</details>
<details> <summary><strong>Inspect Data Structure</strong></summary>
  
```python  
df_payment.head()
df_product.head()
df_transactions.head()
```
- Confirms column names & structure
- Identifies unexpected nulls or formatting issues early
</details>
<details> <summary><strong>Merge Payment + Product Dataset</strong></summary>

Purpose: Enrich payment data with product attributes using product_id.
```python  
df_payment_enriched = pd.merge(
    df_payment,
    df_product,
    on='product_id'
)

df_payment_enriched.head()
```
</details>
<details> <summary><strong>Inspect Data Structure</strong></summary>
  
```python  
df_payment_enriched.info()
df_transactions.info()
```
- Checked schema, datatypes, and memory usage
- Identified missing values and inconsistent types
</details>
<details> <summary><strong>Check Missing Data</strong></summary>

df_transactions — Missing Overview
- sender_id: 49,059 missing (3.7%) → Ignore
- receiver_id: 164,795 missing (12.4%) → Ignore
- extra_info: 1.3M missing (99.5%) → Ignore
```python 
df_transactions.head()
```
</details>
<details> <summary><strong>Fix Incorrect Data Types</strong></summary>
  
*Remove unused column*
```python
if 'date' in df_transactions.columns:
    df_transactions = df_transactions.drop(columns=['date'])
```
*Convert timestamp & ID fields*
```python
# Convert timestamp
df_transactions['timeStamp'] = pd.to_datetime(
    df_transactions['timeStamp'], 
    unit='ms', 
    errors='coerce'
)

# Convert sender/receiver IDs
df_transactions['sender_id'] = df_transactions['sender_id'].astype('int64')
df_transactions['receiver_id'] = df_transactions['receiver_id'].astype('int64')

# Convert report_month
df_payment_enriched['report_month'] = pd.to_datetime(df_payment_enriched['report_month'])
```
</details>
<details> <summary><strong>Re-Validate Data Types</strong></summary>
  
```python
df_transactions.info()
df_payment_enriched.info()
```
</details>
<details> <summary><strong>Handle Duplicates</strong></summary>

```python
df_transactions = df_transactions[~df_transactions.duplicated()]
df_transactions.duplicated().sum()   # Output: 0
```
- Removed 25 duplicate rows
</details>
<details> <summary><strong>Detect Outliers & Invalid Values</strong></summary>

*Summary Statistics*
```python  
df_payment_enriched.describe()
df_transactions.describe()
```
*Invalid Value Found*
- receiver_id = -63 → invalid → Action: Remove
```python  
df_transactions = df_transactions[df_transactions['receiver_id'] != -63]
df_transactions.describe()
```
</details>
<details> <summary><strong>Cleaning Summary</strong></summary>

| Issue Found                 | Action Taken                 |
| --------------------------- | ---------------------------- |
| Missing sender/receiver IDs | Ignored                      |
| `extra_info` 99% null       | Ignored                      |
| Wrong data types            | Fixed (datetime, ID casting) |
| Duplicate rows              | Removed                      |
| Outlier receiver_id = -63   | Removed                      |
| Date column formats         | Standardized                 |

Dataset is now fully clean, validated, and ready for Analysis.
</details>

---

## 🧠 4. Analysis & Insights

### 🔍 **A. Top 3 Products by Total Volume**
[In]:
```python
# Top 3 product_ids with highest total volume
product_with_volume = df_payment_enriched.groupby("product_id")["volume"].sum()
product_with_volume.sort_values(ascending=False).head(3)
```
<details>
  <summary>[Out]:</summary>

<img width="319" height="252" alt="image" src="https://github.com/user-attachments/assets/ec1af39a-0b42-4e2a-83b5-e875819ec8b9" />
</details>

**Insight:**  
- A small number of products generate disproportionately high volume.  
- Product **429** leads by a large margin → indicating high user demand or bundle usage.  
- Product concentration risk exists — business should diversify revenue sources.
  
---

### 🔍 B. Validate Product Ownership Rules
[In]:
```python
# Check if any product is owned by more than 1 team
product_with_team = df_product.groupby('product_id')['team_own'].nunique()
product_with_team.sort_values(ascending=False).head()
```
<details>
  <summary>[Out]:</summary>

<img width="272" height="353" alt="image" src="https://github.com/user-attachments/assets/937c74f8-fac6-41a7-ad2f-c87bfd09149e" />
</details>

**Insight:**  
- Each product should belong to exactly 1 team.
- Validation shows all product_ids are owned by a single team → clean governance and no ownership conflict.

---

### 🔍 C. Lowest-Performing Team Since Q2 2023
[In]:
```python
# Lowest performing team since Q2 2023
filter_data = df_payment_enriched[df_payment_enriched['report_month'] >= pd.Timestamp('2023-04-01')]
team_with_volume = filter_data.groupby('team_own')['volume'].sum()
team_with_volume.sort_values(ascending=True).head(1)
```
<details>
  <summary>[Out]:</summary>

<img width="274" height="159" alt="image" src="https://github.com/user-attachments/assets/382d8275-7c21-42bd-8cd3-12bc7d61b3cc" />
</details>

**Insight:** 
- Filtering from 2023-04-01 onward, Team APS shows the lowest total volume.
- This indicates possible operational issues, product decline, or low customer engagement since Q2.
- Requires performance review and product strategy adjustment.

---

### 🔍 D. Refund Transactions — Top Source Contributor

# Find which source_id contributes the most to refunds
[In]:
```python
filter_payment_group = df_payment[df_payment['payment_group'] == 'refund']
source_with_volume = filter_payment_group.groupby('source_id')['volume'].sum()
source_with_volume.sort_values(ascending=False).head(1)
```
<details>
  <summary>[Out]:</summary>

<img width="343" height="158" alt="image" src="https://github.com/user-attachments/assets/19bddd0f-1f55-45ca-adad-c9a79c6f58c6" />
</details>

**Insight:** 
- Refunds are highly concentrated in Source ID 38, contributing the largest refund volume.
  *This may signal issues with:*
- A specific merchant integration
- Product bugs
- Payment partner API
- Requires immediate root-cause analysis.

---

### 🔍 E. Transaction Type Classification
[In]:
```python
#Define type of transactions (‘transaction_type’) for each row
def trans_type(trans):
  if trans.transType == 2 and trans.merchant_id == 1205:
    return "Bank Transfer Transaction"
  if trans.transType == 2 and trans.merchant_id == 2260:
    return "Withdraw Money Transaction"
  if trans.transType == 2 and trans.merchant_id == 2270:
    return "Top Up Money Transaction"
  if trans.transType == 2 and trans.merchant_id != 1205 and trans.merchant_id != 2260 and trans.merchant_id != 2270:
    return "Payment Transaction"
  if trans.transType == 8 and trans.merchant_id == 2250:
    return "Transfer Money Transaction"
  if trans.transType == 8 and trans.merchant_id !=2250:
    return "Slit Bill Transaction"
  else:
    return "Invalid Transaction"
    
df_transactions['trans_type'] = df_transactions.apply(trans_type, axis=1)
df_transactions.head()
```
<details>
  <summary>[Out]:</summary>

<img width="1736" height="296" alt="image" src="https://github.com/user-attachments/assets/7439c744-9a11-46fd-b026-ffbc6c517297" />
</details>

**Insight:** 
- Transaction rules were applied to classify: Bank Transfer, Payment, Withdraw, Top Up, Transfer, Split Bill.
- Classification is essential for understanding channel behavior and detecting invalid transactions.
  
---

### 🔍 🔍 F. Transaction Metrics by Type
[In]:
```python
# Volume, count, sender, receiver metrics by transaction type (excluding invalid)
df_transactions = df_transactions[df_transactions['trans_type'] != 'Invalid Transaction']
df_transactions.groupby('trans_type').agg({
    'transaction_id':'count',
    'volume':'sum',
    'sender_id':'count',
    'receiver_id':'count'
})
```
<details>
  <summary>[Out]:</summary>

<img width="991" height="386" alt="image" src="https://github.com/user-attachments/assets/1a4a6bdd-ab4c-4514-bfe8-1b3289592375" />
</details>

**Insight:** 
- Payment and Top Up transactions dominate both in count and volume.
- Split Bill has the smallest footprint → potential area for targeted growth.
- High volumes in Transfer indicate strong user peer-to-peer engagement.

---

## ⭐ 5. Key Findings Summary

### 1. Top 3 Best-Performing Products (by Volume)
- Product 429: 14.66B volume
- Product 372: 13.71B volume
- Product 431: 7.82B volume
➡️ These products drive the majority of platform income.

### 2. Product Ownership Check
- All products are owned by exactly 1 team.
➡️ No organizational conflict in product ownership.

### 3. Lowest-Performing Team Since Q2 2023
- Team APS has the lowest total volume (~27.29B).
- Category PXXXXXB contributes 0 volume to this team.
➡️ Indicates severe underperformance and unused product/category.

### 4. Refund Transaction Contributors
- For all refund transactions, source_id = 38 contributes the highest refund volume (36.5B).
➡️ This source should be investigated — potential system, merchant, or risk issue.

### 5. Transaction Breakdown by Type
| Transaction Type | # Transactions | Total Volume |
| ---------------- | -------------- | ------------ |
| Payment          | 260,332        | 343.85B      |
| Transfer Money   | 341,173        | 370.33B      |
| Top Up           | 290,498        | 108.60B      |
| Withdraw         | 33,725         | 23.41B       |
| Bank Transfer    | 14,004         | 100.61B      |
| Split Bill       | 1,376          | 4.9M         |

---

## 🧭 6. Recommendations

Based on the analysis of user behavior, product performance, team ownership, and transaction patterns, the following recommendations are proposed:

### **1. Product Performance Optimization**
- **Prioritize high-volume products** (e.g., Product IDs 429, 372, 431) by allocating more marketing, promotional bundles, and operational support.
- **Investigate low-volume products** to identify pricing issues, lack of visibility, or placement problems.
- **Enhance product lifecycle management** by monitoring product trends monthly to detect early performance drops.

### **2. Improve Team Operational Efficiency**
- **Team APS shows the lowest performance in Q2 2023** → conduct operational review on workload, staffing, or process bottlenecks.
- **Investigate categories with zero contribution** (e.g., PXXXXXB), ensuring no missing configurations or inactive product listings.
- Establish **performance KPIs per team** such as volume contribution, product coverage, and refund interactions.

### **3. Strengthen Refund & Risk Controls**
- **Source ID 38 contributes ~60% of refund volume** → requires urgent root-cause analysis:
  - Check merchant/API integration issues  
  - Validate fraud patterns  
  - Review authorization, compliance, and operational logs  
- Implement **automated monitoring** to flag abnormal spikes in refund transactions at real-time.

### **4. Enhance Transaction Journey & User Experience**
- **Payment and Top-Up transactions have the highest engagement** → prioritize UX improvements, reduce steps, and improve success rate.
- **Split Bill transactions show low volume** → consider:
  - better visibility in app  
  - incentives for social or group payments  
  - simplified UI/UX  
- Investigate invalid transactions and ensure **validation rules** prevent incomplete or malformed transaction requests.

### **5. Strengthen Sender & Receiver Ecosystem**
- High-volume sender and receiver IDs show concentration → evaluate if:
  - certain merchants or corporate partners dominate usage  
  - risk of dependency exists  
- Expand adoption across new user segments to diversify usage and reduce systemic risk.

---

## 🛠️ 7. Tools & Skills Demonstrated

- **Python:** pandas, numpy, seaborn, matplotlib
- **Data Cleaning:** missing value handling, data type correction, fixing anomalies, removing duplicates
- **Exploratory Data Analysis (EDA):** distribution analysis, aggregation, pattern detection
- **Product Performance Analysis:** top sellers, category contribution, team ownership validation
- **User & Transaction Behavior Analysis:** sender/receiver engagement, transaction-type classification
- **Payment & Refund Analytics:** refund contributor detection, failure pattern identification
- **📈 Trend & Volume Analysis:** time-based filtering (Q2), performance comparison
- **🧠 Business Insights & Recommendations:** operational improvements, product focus, risk detection
