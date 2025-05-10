# Customer Segmentation using Advanced RFMT & CFMT Analysis on Online Retail Data

## Project Overview

This project performs a comprehensive customer segmentation analysis on a transnational online retail dataset spanning three years (2009, 2010, 2011). The primary goal is to identify distinct customer groups based on their purchasing and cancellation behaviors using advanced feature engineering techniques like RFMT purchases (Recency, Frequency, Monetary, Interpurchase Time (average time between a customer's purchases)) analysis and CFMT Cancellation (CRecency, CFrequency, CMonetary, CTime (average time between cancellations)) . The segmentation is achieved using the K-Means clustering algorithm, and the results are visualized and made interactive through a Power BI dashboard.

This project showcases a meticulous data preprocessing pipeline, thoughtful feature engineering, robust model selection, and insightful cluster interpretation, demonstrating an advanced approach to customer analytics.

## Dataset

* **Source:** [Online Retail II Data Set on Kaggle](https://www.kaggle.com/datasets/lakshmi25npathi/online-retail-dataset)
* **Description:** This dataset contains all transactions occurring for a UK-based and registered, non-store online retail between 01/12/2009 and 09/12/2011. The company mainly sells unique all-occasion gift-ware, and many of its customers are wholesalers.
* **Original Size:** 1,067,371 transactions with 8 columns.
* **Key Attributes:**
    * `InvoiceNo`: Invoice number (char 'c' indicates cancellation).
    * `StockCode`: Product code.
    * `Description`: Product name.
    * `Quantity`: Quantity per transaction.
    * `InvoiceDate`: Transaction date and time.
    * `UnitPrice`: Price per unit in sterling (£).
    * `CustomerID`: Unique customer identifier.
    * `Country`: Customer's country.

## Project Workflow

The project followed a systematic approach:

1.  **Data Loading & Initial Exploration:**
    * Imported necessary Python libraries (Pandas, NumPy, Scikit-learn, etc.).
    * Loaded the two Excel sheets (`Year 2009-2010` and `Year 2010-2011`) and merged them into a single DataFrame.
    * Performed an initial assessment: identified 1,067,371 transactions, noted missing values in `Description` (4,382) and `CustomerID` (243,007), and detected 34,335 duplicate rows.

2.  **Rigorous Data Preprocessing:**
    * **Handling Missing Values:**
        * `CustomerID`: Rows with missing `CustomerID` were removed, as customer identification is crucial for segmentation. Attempts to impute based on shared `InvoiceNo` were unsuccessful.
        * `Description`: Missing `Description` values corresponding to removed `CustomerID` rows were consequently dropped.
    * **Handling Duplicates:** Duplicate rows were addressed to ensure data quality.
    * **Handling Canceled Transactions:**
        * Identified transactions with `InvoiceNo` starting with 'C'.
        * Matched cancellations to previous purchases based on `CustomerID`, `StockCode`, purchase date < cancellation date, canceled quantity ≤ purchased quantity, and a 60-day return window.
        * **Decision:** Kept these transactions as they provide valuable insights into customer return behavior and retention, crucial for CFMT analysis.
    * **Handling Negative and Zero Values:**
        * `Quantity`: Negative values (returns/cancellations) were kept. Zero values were checked.
        * `Price`: Negative prices were deemed invalid. Zero prices were investigated and kept as they likely represent promotions or bundles.
    * **Outlier Handling:**
        * **Advanced Approach:** Instead of removing outliers, capping was applied to `Quantity` and `Price` columns. This preserves data records while mitigating the disproportionate influence of extreme values, acknowledging that every transaction can hold potential insights.
    * **Data Type Conversion:**
        * `InvoiceNo` and `StockCode` kept as strings.
        * `Quantity` converted to `int64`.
        * `CustomerID` converted to `Int64`.
        * `InvoiceDate` confirmed as datetime.

3.  **Feature Engineering - RFMT & CFMT Models:**

    * **RFMT Analysis (Recency, Frequency, Monetary, Interpurchase_Time):**
        * `Recency (R)`: Days since the customer's last purchase.
        * `Frequency (F)`: Total number of distinct purchases.
        * `Monetary (M)`: Total revenue generated.
        * `Interpurchase Time (T)`: Average time between a customer's purchases. Calculated as `(Last Purchase Date - First Purchase Date) / (Frequency - 1)`. `NaN` for single-purchase customers. (I just care about customer with more than one Frequency so I filtered the data)
        * **Important Note:** Canceled transactions were deliberately excluded from RFMT calculations to reflect true purchasing behavior.

    * **CFMT Model (Cancellation Behavior):**
        * `CRecency (CR)`: Days since the customer's last cancellation.
        * `CFrequency (CF)`: Total number of cancellation transactions.
        * `CMonetary (CM)`: Total revenue lost due to cancellations.
        * `CTime (CT)`: Average time between a customer's cancellations.

4.  **Scoring and Normalization (Bucketing):**
    * RFMT and CFMT metrics were converted into categorical scores (typically 1 to 4) to allow for standardized comparison and clustering.
    * **RFMT Scoring Logic:**
        * Recency: More recent → Higher score.
        * Frequency: More purchases → Higher score.
        * Monetary: Higher spending → Higher score.
        * Interpurchase Time: Shorter time between purchases → Higher score.
    * **CFMT Scoring Logic:**
        * CRecency: More recent cancellations → Lower score (e.g., 1), less recent → Higher score (e.g., 4).
        * CFrequency: More frequent cancellations → Lower score (e.g., 1), less frequent → Higher score (e.g., 3).
        * CMonetary: Higher refund amounts → Lower score (e.g., 1), lower refunds → Higher score (e.g., 4).
        * CTime: Longer time between cancellations → Higher score (e.g., 3), shorter time → Lower score (e.g., 1).

5.  **Data Merging & Final Preparation:**
    * RFMT and CFMT scores were merged based on `CustomerID`.
    * Missing CFMT values (for customers with no cancellations) were filled with the highest score (e.g., 4), indicating optimal behavior in terms of cancellations.

6.  **Customer Segmentation with K-Means Clustering:**
    * The combined RFMT + CFMT scores were used as features for K-Means.
    * **Optimal k Selection:** Evaluated using Silhouette Score and Davies-Bouldin Score:
        * k=3: Silhouette = 0.305, Davies-Bouldin = 1.308
        * k=4: Silhouette = 0.295, Davies-Bouldin = 1.452
        * k=5: Silhouette = 0.281, Davies-Bouldin = 1.355
        * k=6: Silhouette = 0.293, Davies-Bouldin = 1.325
    * **Decision:** `k=3` was chosen as the optimal number of clusters.
    * The model was trained, and customers were assigned to one of three clusters.

7.  **Cluster Analysis & Interpretation:**
    * **Cluster 0: Balanced & Satisfied Customers (1,761 customers)**
        * RFMT: Good scores (R=3.15, F=1.91, M=3.01, T=2.93).
        * CFMT: Very high scores (CR–CT ~3.6–4.0) → low cancellation risk.
        * *Profile:* Reliable, moderately engaged, satisfied, and stable with minimal issues.
    * **Cluster 1: High-Value & Frequent Flyers (1,427 customers)**
        * RFMT: Highest scores (R=3.21, F=2.52, M=3.62, T=3.39) → recent, frequent, high-spending, short purchase intervals.
        * CFMT: Moderate scores (CR–CT ~1.77–2.97) → higher cancellation tendency but still very valuable.
        * *Profile:* Most profitable and active, high engagement, room to reduce cancellations.
    * **Cluster 2: Low Engagement & Potential At-Risk (2,693 customers)**
        * RFMT: Lowest scores (R=1.71, F=1.07, M=1.57, T=2.30) → least recent, infrequent, low spenders, long purchase intervals.
        * CFMT: High scores (CR–CT ~3.78–4.00) → low cancellation rates, potentially due to fewer overall interactions or general indifference.
        * *Profile:* At risk due to poor engagement and low activity. Low cancellations might indicate disengagement rather than satisfaction.

8.  **Visualization & Dashboarding:**
    * Principal Component Analysis (PCA) was used to visualize cluster separation in 2D.
    * Heatmaps were likely used to examine feature distributions across clusters.
    * The final clustered data (`RFMT_CFMT_Clusters.xlsx`) was prepared for Power BI.
        * NaN values in `CFrequency` and `CMonetary` (for customers with no cancellations) were replaced with 0.
    * A Power BI dashboard was developed to interactively explore the clusters.

## Technologies Used

* **Programming Language:** Python 3.x
* **Key Python Libraries:**
    * Pandas: Data manipulation and analysis.
    * NumPy: Numerical computations.
    * Scikit-learn: K-Means clustering, PCA, evaluation metrics.
    * Matplotlib/Seaborn (implied): Data visualization within the notebook.
* **BI Tool:** Microsoft Power BI: Interactive dashboard development.
* **Environment:** Google Colab / Jupyter Notebook.

## Project Structure
.
* Online_Retail_Dataset.xlsx    # The original dataset file (or specific name used)
* Customer_Segmentation_Analysis.ipynb # Jupyter Notebook with all analysis steps
* RFMT_CFMT_Clusters.xlsx       # Processed data with RFMT, CFMT scores and cluster labels
* Customer_Segmentation_Dashboard.pbix # Power BI dashboard file
* README.md                       # This file

## Run the Analysis:**
    * Open `Customer_Segmentation_Analysis.ipynb` in Jupyter Notebook or Google Colab.
    * Ensure the dataset `Online_Retail_Dataset.xlsx` is in the correct path or update the path in the notebook.
    * Execute the cells sequentially to reproduce the analysis.

## View the Dashboard:**
    * Open `Customer_Segmentation_Dashboard.pbix` using Microsoft Power BI Desktop.
    * The dashboard is designed to use `RFMT_CFMT_Clusters.xlsx` as its data source. Ensure Power BI can access this file, or update the data source path within Power BI if needed.

## Power BI Dashboard Highlights

The interactive Power BI dashboard provides a dynamic way to explore the customer segments. Key features include:

* **Cluster Filter:** Allows users to select and view data for a specific customer cluster (Cluster 0, 1, or 2).
* **Key Performance Indicators (KPIs) Cards:** For the selected cluster (or overall):
    * Total Number of Customers
    * Average Number of Orders
    * Average Interpurchase Time (Gap between orders)
    * Average Monetary Value (Spending)
    * Average Number of Canceled Orders
    * Average Monetary Value of Returns
* **Visualizations:** Charts displaying distributions for the selected cluster:
    * Number of customers per cluster.
    * Customer distribution across Frequency (orders count) groups.
    * Customer distribution across Interpurchase Time (time gap) groups.
    * Customer distribution across Monetary (spending) groups.
    * Customer distribution across Cancellation Frequency (orders canceled count) groups.
    * Customer distribution across Cancellation Monetary (value returned) groups.
    *(These "groups" refer to categorical bins created from the continuous RFMT/CFMT metrics for clearer visualization).*

## Conclusion

This project successfully segments customers into three distinct groups using a robust K-Means clustering approach based on advanced RFMT and CFMT features. The detailed cluster profiles offer actionable insights for targeted marketing campaigns, personalized customer experiences, and strategic retention efforts. The Power BI dashboard serves as a powerful tool for stakeholders to explore and understand these customer segments dynamically.

## Author & Contact

* **Name:** [Engy Saeed]
* **Email:** `engysead498@gmail.com`
* **LinkedIn:** `https://www.linkedin.com/in/engy-saeed-b47784276?lipi=urn%3Ali%3Apage%3Ad_flagship3_profile_view_base_contact_details%3Bh4ZG9v%2BhTR2sN8YzYBFy6Q%3D%3D`
