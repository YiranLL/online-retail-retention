### 1. Information about the dataset
This project uses _UCI Online Retail II_ dataset with the transaction records of an UK online retail between 01/12/2009 and 09/12/2011. The company mainly sells unique all-occasion gift-ware. Many customers of the company are wholesalers. The original Excel file contains more than one million transaction lines across two Excel sheets.   
 
Each row represents one purchase, with main columns including:  
- Invoice number
- Stock code and description
- Invoice date and time
- Purchase quantity
- Price of the product
- Customer ID
- Country  
   
This dataset includes various real life data quality issues, including cancellation of purchase, returns, negative quantities and prices, missing customer ID, duplicate records and administrative transaction records.  

After proper processing of the data, the merchandise transaction lines were aggregated into 36,594 valid orders, belonging to 5,852 identifiable customers. After excluding 596 customers without a complete 90-day observation window, the final data frame contains 5,256 eligible customers.  

### 2. Problem
This project aims to answer the following question:  

> Using only the information available when a customer completes their first valid merchandise order, can we estimate whether they will place another valid order within 90 days?  

The observation unit is per customer, and the prediction point is the completion of the first valid order.
  
We define the dependent variable as:
- `1` if the customer places at least one valid merchandise order within 90 days strictly after the first order.
- `0` if no further valid order is placed within 90 days strictly after the first order.

Customers whose first orders happens within the 90-days period before the end of the dataset were excluded, because there are insufficient data to determine whether they place valid later order within 90 days strictly after the first order.

The intended business application is to rank the customer based on their estimated likelihood of repeat-purchase and identify priority group of customers.
### 3. Approach

The analysis consists of four stages.
#### 3.1 Data quality check and cleaning

First, we combined two Excel worksheet, then we examined the following:
- Missing values
- Cancelled purchase
- Returns
- Invalid quantity and price
- Repeated record

For each transaction, we computed the total value.
The administrative transaction code are excluded from the merchandise transactions.

#### 3.2 Aggregation of purchases and invoices

Multiple purchases that belong to the same invoice are aggregated into one order, each order contains the following variables:
- Total value of the products
- Total quantity of the products
- Quantity of product types
- Average price per item
- Order timestamp
- Country

Then, we sorted the  orders per customer based on time. The first order is identified as the prediction time, any valid order placed strictly later than the first order is identified as the second order. 

Modelling feature for each customer includes:
- Value of first order
- Total quantity of first order
- Quantity of product types of first order
- Average price per item of first order
- First order timestamp
- Country
Initially, we also included the Month, but subsequent analysis indicates that this feature significantly lowered the performance of the model, hence this feature is removed.

Customer ID, invoice numbers and all transactions after first order are not considered as model feature to avoid data leakage.

#### 3.3 Exploratory Data Analysis
First we calculated the overall 90 day repeated rates and its Wilson confidence interval, then we divided the customers into 5 groups based on value of first order. Each group contains 20% of customers, to compare the trend of repeated rate.

Then we also excluded customers with top 1% value of first order, to check if extreme data significantly impact the overall result.

#### 3.4 Modelling and evaluation

We split the customers chronologically:
- 4,239 earlier customers for training
- 558 subsequent customers for validation
- 459 latest customers for final testing

We use chronological validation instead of random validation because repeated rates in different months are significantly different and has no clear temporal trends, as shown in the following figure.

![[repeat_rate_by_cohort.png]]

Chronological validation simulates real life deployment.

This project compare three models:
- logistic regression
- shallow decision tree
- histogram gradient-boosted trees

The models were evaluated using the following metrics:
- ROC-AUC
- PR-AUC
- log loss
- Brier score
- performance at a fixed 10% targeting capacity
Permutation importance was used to assess feature contributions on the validation period. Bootstrap resampling was used to quantify uncertainty around the final test results.
### 4. Key findings

Among the 5,256 eligible customers, 2,445 placed another valid order within 90 days. The estimated 90-day repeated rate is 46.52%, with a 95% Wilson confidence interval of \[45.17%, 47.87%\].

Value of first-order, quantity and products were strongly right-skewed. Most customers placed moderate-sized orders, while a small number placed exceptionally large orders. The mean first-order value was therefore substantially higher than the median.

Repeat purchasing increased monotonically across first-order-value quintiles:
- 34.98% in the lowest-value group
- 57.18% in the highest-value group

Repeat purchasing also increased across product-diversity groups:
- 40.37% in the lowest-diversity group
- 54.32% in the highest-diversity group

Repeat rates varied substantially across month cohorts. The training repeat rate was 47.53%, the validation repeat rate was 35.30%, and the test repeat rate was 50.76%. This indicates temporal instability in the purchases.

Excluding the highest 1% of first-order values changed the overall repeat rate only from 46.52% to 46.26%. The aggregate repeated rate estimate was therefore not significantly affected by extreme first orders.

Logistic regression produced a validation ROC-AUC of 0.530 and PR-AUC of 0.364. A shallow decision tree achieved a similar ROC-AUC but improved PR-AUC to 0.380. The initial boosted-tree model performed better, with a validation ROC-AUC of 0.567 and PR-AUC of 0.393.

Permutation importance identified first-order value as the strongest validation feature, followed by quantity and number of unique products. Calendar month had negative validation importance, suggesting that historical seasonal patterns did not transfer reliably to later customers.

After removing month feature, validation performance improved to:
- ROC-AUC: 0.601;
- PR-AUC: 0.463;
- log loss: 0.660;
- Brier score: 0.234.

At a 10% validation capacity, the selected group had a repeat rate of 55.4%, compared with 35.3% overall. This corresponded to a validation lift of 1.57.

The boosted-tree model without month was selected as the final model with the final test results shown as the following:
- ROC-AUC: 0.562;
- PR-AUC: 0.569;
- baseline PR-AUC: 0.508;
- top-10% repeat rate: 58.7%;
- overall test repeat rate: 50.8%;
- top-10% lift: 1.16.

The model ranked customers modestly better than by randomness. However, its test log loss and Brier score were slightly worse than the constant-probability baseline. Its scores are therefore more useful for relative customer ranking than as precise individual probabilities.

The 95% bootstrap confidence interval for test ROC-AUC was approximately \[0.511, 0.614\]. The top-10% lift interval was \[0.90, 1.47\]. Because the lift interval contains one, the available test sample did not provide strong evidence that the observed lift would consistently remain positive.

### 5. Limitation

Approximately 23% of raw transaction lines lacked customer ID and could not be included in customer-level repeated rate analysis. Because these transactions differed from identified transactions, the results may not generalise to anonymous customers.

Returns and cancellations were excluded when constructing positive merchandise orders. Consequently, the outcome measures repeat ordering rather than whether the merchandise was ultimately retained or generated positive net revenue. A customer could be classified as retained even if the original or repeat purchase was later returned.

The United Kingdom was the only country with at least 100 eligible customers, so reasonable country-level comparison was not possible.

Several features are correlated, including order value, quantity, unique products and product lines. Thus, feature importance cannot be interpreted as an independent.

Customer behaviour and outcome changed substantially across time. Although removing calendar month improved validation results, a noticeable training–validation performance gap remained.

The model predicts which customers are more likely to repeat naturally. It does not identify which customers would change their behaviour because of a discount, reminder or other intervention.

### 6. Further development

A future version of this project could take the returns in accounts to establish outcomes such real repeated purchases. 

Product descriptions could be used to create interpretable categories through keywords rule, text embeddings or clustering. These categories could be tested as first order features, and could identify which category of product contributes to repeated rates.

The modelling process could be extended through time-series bakctest. This could provide a more reliable estimate of temporal stability than a single validation and test split.

The recommended business next step is a randomised retention experiment. Customers could be divided into broad predicted repeat groups, with treatments such as discounts and promotions randomised within each group. The primary outcome would be 90-day repeat purchasing, while net revenue, gross margin, cancellations, returns and unsubscribe rates would serve as guardrails.