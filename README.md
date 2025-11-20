# 💳 User Behavior, Transaction Patterns & Refund Insights | E-Wallet Analytics | Python
<img width="1576" height="854" alt="image" src="https://github.com/user-attachments/assets/b0ffee51-2fba-4cd0-95f7-426ab78b5481" />

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

**🎯Project Outcome:**

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
  <summary>Table 1: Products</summary>

| Column Name  | Data Type | Description |
|-------------|----------|-------------|
| product_id  | INT      | Unique identifier for each product |
| category    | TEXT     | Product category |
| team_own    | TEXT     | Team responsible for the product |
</details>
<details>
  <summary>Table 2: Payment Report</summary>

| Column Name    | Data Type | Description |
|---------------|----------|-------------|
| report_month  | DATE     | Month of the payment report |
| payment_group | TEXT     | Type of payment (e.g., refund, purchase) |
| product_id    | INT      | Associated product ID |
| source_id     | INT      | Source of the transaction |
| volume        | FLOAT    | Total payment volume |
</details>
<details>
  <summary>Table 3: Transactions</summary>

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

Dataset is now fully clean, validated, and ready for EDA + Analysis.
</details>

---

## 🧠 4. Analysis & Insights

### 🔍 **A. Product Performance Insights**
[In]:

```python
# Calculate volume by product_id
volume_by_product = payment_product.groupby('product_id')['volume'].agg('sum').reset_index()

# Sort volume in descending order
volume_by_product.sort_values(by='volume', ascending=False)

# Filter the top 3 product_ids with the highest volume
top_3_productid = volume_by_product.head(3)

# Print the results
print("Top 3 product_ids with the highest volume:")
print(top_3_productid)
```

[Out]:

![Image](https://github.com/user-attachments/assets/bc5a3949-cd1d-4abb-8bfd-5a71278da524)

**Insight**
- **Product ID 15** and **Product ID 12** generate the highest volume, making them top drivers of platform activity.  
- **Product ID 3** shows significantly lower volume → possible low demand or operational issues.  
- High-volume products should be prioritized for **marketing**, **inventory**, and **user targeting** strategies.


### 🔍 **B. Team Ownership & Operational Integrity**
- Every product is managed by **one unique team**, indicating clear ownership and no conflicts.  
- No overlapping responsibilities → efficient structure and reduced risk of duplicated work.  
- Teams show stable management discipline, supporting smooth operational processes.

---

### 🔍 **C. Team Performance (Q2 2023 Onwards)**
- **Team APS** records the **lowest performance** in total transaction volume since Q2.2023.  
- Its weakest category, **PXXXXXB**, contributes **0 volume**, indicating either inactive product lines or operational blockers.  
- APS may need **resource support**, **workflow review**, or **product strategy realignment**.

---

### 🔍 **D. Refund Behavior & Source Contributions**
- Refunds are heavily driven by a single source: **Source ID 38**, contributing **~59%** of all refund volume.  
- This source likely indicates a **system issue**, **merchant problem**, or **fraud risk**.  
- Reducing refund volume from this source can substantially improve platform stability.

---

### 🔍 **E. Transaction Type Distribution**
- **Payment** and **Top Up** transactions dominate the platform with the highest volume.  
- **Split Bill** has the lowest engagement → niche use case but potential for growth with promotion.  
- Understanding each type’s sender/receiver counts helps identify user engagement patterns across transaction flows.

---

### 🔍 **F. User Interaction Across Transaction Types**
- High sender/receiver counts in **Payment** and **Top Up** indicate strong user reliance on core wallet functions.  
- **Withdraw** and **Transfer** show moderate activity, suggesting selective usage scenarios.  
- Engagement indicators highlight where to invest UX improvements and marketing.

---

## ⭐ 5. Key Findings Summary

- Weekend/evening periods drive majority user activity.  
- In-app and QR payments outperform web transactions.  
- Top merchants and top users generate most revenue.  
- Failure and refund patterns are **not random** and cluster in specific areas.  
- High-top-up users offer strong monetization potential.  
- There is significant opportunity to improve **conversion**, **retention**, and **operational efficiency**.

---

## 🧭 6. Recommendations

### **1. Reduce Transaction Failures**
- Optimize bank APIs and authentication flows.  
- Promote wallet-balance payments (highest success rate).

### **2. Improve Merchant Quality**
- Work with high-refund merchants to reduce errors/order issues.  
- Boost promotions for top-performing merchants.

### **3. Enhance Payment Channels**
- Improve **web checkout** to reduce drop-offs.  
- Expand QR code adoption due to strong usage growth.

### **4. Increase User Engagement**
- Introduce reward programs for active users.  
- Target high top-up users with personalized offers.

### **5. Strengthen Fraud & Anomaly Detection**
- Monitor irregular refund/transaction patterns.  
- Auto-flag merchants with unusual failure spikes.

---

## 🛠️ 7. Tools & Skills Demonstrated

- **Python:** pandas, numpy, seaborn, matplotlib  
- **EDA:** data cleaning, anomaly detection, feature exploration  
- **Customer segmentation**  
- **Merchant performance analytics**  
- **Refund & failure pattern detection**  
- **Trend analysis & storytelling**  
- **Business recommendations**

---
> 📌 *This project highlights end-to-end analytics skills: from raw data → insights → business recommendations.*



