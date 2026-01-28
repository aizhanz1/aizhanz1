# RFM-Customer-Segmentation (Online Retail)

## Context
E-commerce businesses generate thousands of transactions, but not all customers behave the same way. Some customers buy frequently, some spend a lot but rarely, and some disappear after one purchase.

To make marketing more efficient (targeting, retention, win-back campaigns), we can group customers using **RFM analysis** — a classic segmentation method based on:

- **Recency**: how recently a customer purchased  
- **Frequency**: how often they purchase  
- **Monetary**: how much they spend  

This project builds an **RFM model** using the Online Retail transactional dataset and produces customer segments that can be used for lifecycle marketing.

---

## Objective
Using the transaction data, the goal is to:

- Create an **RFM table** (Recency, Frequency, Monetary) per customer
- Assign **R/F/M scores** using quantile-based bucketing
- Create an **RFM Score / Index** for each customer
- Generate actionable **customer segments** (e.g., Champions, Loyal, At Risk)
- Provide **business recommendations** for each segment

---

## Data Description
Dataset file: **Online_Retail.xlsx** (sheet: `Online Retail`)

**Columns:**
- **InvoiceNo**: Unique invoice number (in some cases begins with “C” for cancellation)
- **StockCode**: Product code
- **Description**: Product description
- **Quantity**: Units purchased
- **InvoiceDate**: Date and time of transaction
- **UnitPrice**: Price per unit
- **CustomerID**: Customer identifier
- **Country**: Customer country

---

## Data Preparation
To build a clean RFM dataset, we applied common transaction-cleaning rules:

**Removed:**
- Cancellations: invoices where `InvoiceNo` starts with **"C"**
- Returns / invalid rows: `Quantity <= 0` or `UnitPrice <= 0`
- Missing customer IDs: `CustomerID is null`

**Created feature:**
- **TotalSales = Quantity × UnitPrice**

**Data coverage (after cleaning):**
- Transactions used: **397,884**
- Unique customers: **4,338**
- Date range: **2010-12-01 → 2011-12-09**

---

## RFM Model Logic

### 1) Snapshot Date
We define a reference point (snapshot date) to compute recency:

- **Snapshot date = max(InvoiceDate) + 1 day**
- Here: max InvoiceDate is **2011-12-09**, so snapshot is **2011-12-10**

---

### 2) RFM Definitions
RFM metrics are computed per `CustomerID`:

- **Recency** = days since last purchase  
  `Recency = (snapshot_date - max(InvoiceDate)).days`

- **Frequency** = number of unique purchases  
  `Frequency = nunique(InvoiceNo)`

- **Monetary** = total revenue generated  
  `Monetary = sum(TotalSales)`

---

### 3) R / F / M Scoring (Quantile Buckets)
We score customers into **3 buckets** (low/medium/high) using quantiles (similar to `qcut`).

**Thresholds used (approx.):**
- **Recency** split points: **25 days**, **91 days**  
  Lower recency is better → higher score
- **Frequency** split points: **1 purchase**, **4 purchases**
- **Monetary** split points: **388.72**, **1197.55**

**Score rules:**
- **R_Score**: lower Recency → higher score (3 is best)
- **F_Score**: higher Frequency → higher score
- **M_Score**: higher Monetary → higher score

---

### 4) RFM Score and Index
- **RFM_Score** = concatenation of R_Score, F_Score, M_Score  
  Example: `333`, `213`, `121`

- **RFM_Index** (numeric)  
  `RFM_Index = R*100 + F*10 + M`

---

## Customer Segmentation
After scoring, customers are grouped into practical segments:

**Segments created:**
- **Champions**: bought recently, buy often, spend high  
- **Loyal Customers**: consistently active shoppers  
- **Potential Loyalists**: good potential, need nurturing  
- **New Customers**: very recent, low frequency  
- **Promising**: somewhat recent, low repeat so far  
- **Needs Attention**: not recent enough, moderate frequency  
- **At Risk**: used to buy often, now inactive  
- **Hibernating**: old activity, low frequency and low spend  

**Segment distribution (customer count):**
- Loyal_Customers: **910**
- Champions: **873**
- Hibernating: **846**
- Potential_Loyalists: **510**
- Needs_Attention: **476**
- Promising: **474**
- New_Customers: **126**
- At_Risk: **123**

---

## Conclusions (Key Insights)
- A large share of customers are **Champions + Loyal**, meaning the business has a strong core customer base worth protecting.
- A similarly large share is **Hibernating**, showing many customers churn after limited engagement.
- Smaller but important groups like **At Risk** represent high ROI opportunities for win-back strategies.

---

## Business Recommendations

### Champions
- Give early access to launches, VIP perks, loyalty bonuses
- Encourage referrals and reviews
- Avoid over-discounting (protect margin)

### Loyal Customers
- Personalized cross-sell bundles
- Subscription / replenishment prompts
- Loyalty points acceleration to keep frequency high

### Potential Loyalists
- Nurture with onboarding sequences and personalized offers
- Triggered campaigns after 2nd purchase to push habit formation

### New Customers
- Welcome journey (email/SMS), product education, best-sellers
- “Second purchase” incentive within 7–14 days

### Promising / Needs Attention
- Reminder campaigns and tailored discounts
- Recommend products based on the first purchase category
- Reactivation within 30–60 days window

### At Risk
- Win-back campaigns with strong incentive and urgency
- Identify previously purchased categories and offer highly relevant deals

### Hibernating
- Low-cost reactivation (broad campaigns)
- If no response → suppress to reduce marketing costs and improve deliverability

---

## Tools & Files
- **Excel dataset:** `Online_Retail.xlsx`
- **Notebook:** `RFM_model (1).ipynb`
- Python stack: `pandas`, `numpy`, `datetime`
