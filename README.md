# 💳 User Behavior, Transaction Patterns & Refund Insights | E-Wallet Analytics | Python

**Author:** Huong Le  
**Domain:** Digital Payments & E-commerce Analytics  
**Tools Used:** Python (Pandas, NumPy, Matplotlib, Seaborn)  

---

## 📌 1. Background & Overview

Digital wallet platforms process large volumes of customer transactions daily. Understanding **user behavior**, **payment success**, and **revenue patterns** is essential for improving customer experience and financial performance.

This project analyzes an **E-Wallet transaction dataset** using Python to uncover:

- Customer and merchant activity patterns  
- Payment success vs failure behavior  
- Refund trends and risk areas  
- High-value users & high-performing merchants  
- Opportunities to improve conversion and revenue  

The goal is to demonstrate end-to-end **data analytics thinking**, turning raw data into insights and business actions.

---

## ❓ 2. Business Questions

1. What are the key **user behavior patterns** across the platform?  
2. Which merchants drive the most **revenue** and **successful transactions**?  
3. What causes **transaction failures**, and how frequent are they?  
4. Which merchants or categories have the **highest refund rates**?  
5. What are the **conversion opportunities** across payment channels?  
6. How can the platform improve **retention, efficiency, and revenue**?

---
## 📂 Dataset Description

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


##### Table 1: Products Table (`product.csv`)

| Column Name  | Data Type | Description |
|-------------|----------|-------------|
| product_id  | INT      | Unique identifier for each product |
| category    | TEXT     | Product category |
| team_own    | TEXT     | Team responsible for the product |


##### Table 2: Payment Report (`payment_report.csv`)

| Column Name    | Data Type | Description |
|---------------|----------|-------------|
| report_month  | DATE     | Month of the payment report |
| payment_group | TEXT     | Type of payment (e.g., refund, purchase) |
| product_id    | INT      | Associated product ID |
| source_id     | INT      | Source of the transaction |
| volume        | FLOAT    | Total payment volume |


##### Table 3: Transactions (`transactions.csv`)

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

---

## ⚒️ Main Process

### 1️⃣ Data Cleaning & Preprocessing 

#### Step 1. Import library

```python
import numpy as np
import matlotblib as plt
import seaborn as sns
import pandas as pd
```

#### Step 2. Load dataset 

```python
#Load data to Colab
from google.colab import drive
drive.mount('/content/drive')

# Import file csv to Colab
import pandas as pd
payment = pd.read_csv('/content/drive/MyDrive/Python_Project2/payment_report.csv', encoding='utf-8')
product = pd.read_csv('/content/drive/MyDrive/Python_Project2/product.csv', encoding='utf-8')
transaction = pd.read_csv('/content/drive/MyDrive/Python_Project2/transactions.csv', encoding='utf-8')
```

#### Step 3. Display the first 5 rows of the product table

```python
payment.head()
product.head()
transaction.head()
```
📝 **Checked for Missing Values**

- Missing values were detected in the **transaction** table:
  - **sender_id**: 49,059 missing values
  - **receiver_id**: 164,795 missing values
  - **extra_info**: 1,317,907 missing values

No missing values were found in the **payment** or **product** tables.

📝 **Checked for Duplicates**

- **Payment table**: No duplicates were found.
- **Product table**: No duplicates were found.
- **Transaction table**: 28 duplicate rows were identified.

### 2️⃣ Exploratory Data Analysis (EDA)

📝 **Handle Missing Values**

```python
# The sender_id and receiver_id columns in the Transaction table are not important, so missing values are replaced with 'Unknow' for consistency
transaction['sender_id'] = transaction['sender_id'].fillna(value='Unknow')
transaction['receiver_id'] = transaction['receiver_id'].fillna(value='Unknow')

# Drop the 'extra_info' column from the Transaction table
transaction.drop('extra_info', axis=1, inplace=True)
```

📝 **Handle Duplicate**

```python

#Drop duplicate
transaction.drop_duplicates(subset=None, keep='first')
```
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



