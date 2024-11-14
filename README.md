# Python-RFM-Analysis-and-Customer-Segmentation-Visualization
## Project Overview
This project implements a comprehensive RFM (Recency, Frequency, Monetary) analysis system for SuperStore's global retail operations. The analysis aims to support the Marketing team's customer segmentation efforts for their year-end marketing campaigns, transitioning from manual Excel-based analysis to an automated Python solution due to increased data volume.
## Business Context
SuperStore, a global retail company, needs to segment their large customer base effectively for targeted Christmas and New Year marketing campaigns. The Marketing department requires a data-driven approach to:
- Identify and reward loyal customers
- Discover potential high-value customers
- Develop targeted marketing strategies for different customer segments
## Key Analysis Areas
### RFM Score Calculation
- Recency (R): Based on InvoiceDate, Days between last purchase and reference date
- Frequency (F): Derived from unique InvoiceNo count per CustomerID
- Monetary (M): Calculated from Quantity × UnitPrice
### Customer Segmentation
Implementation of quintile-based scoring system:
- Score range: 1-5 for each RFM component
- Statistical quintile method for fair distribution
- Combined RFM segmentation based on score patterns
### Visualization & Analysis
- Distribution of Recency, Frequency, Monetary 
- Customer segment distribution visualization of Customer Count, Customer Revenue
## Bussiness value
### Immediate Benefits
#### Marketing Efficiency
- Targeted campaigns based on customer segments
- Reduced marketing costs through precise targeting
- Higher conversion rates from personalized approaches
#### Customer Insights
- Clear identification of VIP customers
- Early detection of churning customers
- Recognition of potential high-value customers
#### Revenue Optimization
- Focused retention strategies for valuable segments
- Customized promotions for each customer group
- Better resource allocation for marketing campaigns
### Long-term Impact
- Improved customer lifetime value
- Enhanced customer loyalty
- Data-driven decision making for future campaigns
## Dataset Description
This is a transnational data set which contains all the transactions occurring between 01/12/2010 and 09/12/2011 for a UK-based and registered non-store online retail. Reference date for Recency calculation: December 31, 2011. The company mainly sells unique all-occasion gifts. Many customers of the company are wholesalers. The analysis uses the following fields:
  
| Field        | Explanation                                                                                              |
|--------------|----------------------------------------------------------------------------------------------------------|
| InvoiceNo    | Invoice number. Nominal, a 6-digit integral number uniquely assigned to each transaction. If this code starts with the letter 'C', it indicates a cancellation. |
| StockCode    | Product (item) code. Nominal, a 5-digit integral number uniquely assigned to each distinct product.      |
| Description  | Product (item) name. Nominal.                                                                            |
| Quantity     | The quantities of each product (item) per transaction. Numeric.                                         |
| InvoiceDate  | Invoice Date and time. Numeric, the day and time when each transaction was generated.                   |
| UnitPrice    | Unit price. Numeric, product price per unit in sterling.                                               |
| CustomerID   | Customer number. Nominal, a 5-digit integral number uniquely assigned to each customer.                  |
| Country      | Country name. Nominal, the name of the country where each customer resides.                             |
## Technical Implementation
- Pandas for data manipulation
- NumPy for numerical computations
- Matplotlib/Seaborn for visualization
## Exploring the Dataset
### Cleaning Data 
```python
# Display general information about the dataframe, including non-null counts and data types
ecommerce_retail.info()

# Provide descriptive statistics for numeric columns such as Quantity and UnitPrice
ecommerce_retail.describe()

# Remove rows with missing values in the CustomerID column to ensure valid customer information
ecommerce_retail = ecommerce_retail.dropna(subset=['CustomerID'])

# Filter rows where Quantity is greater than 0 and drop any rows with missing values in Quantity
# This ensures only transactions with a positive quantity are included
ecommerce_retail = ecommerce_retail[ecommerce_retail['Quantity'] > 0].dropna(subset=['Quantity'])

# Filter rows where UnitPrice is greater than 0 and drop any rows with missing values in UnitPrice
# This keeps only valid transactions with positive prices, important for revenue and profit calculations
ecommerce_retail = ecommerce_retail[ecommerce_retail['UnitPrice'] > 0].dropna(subset=['UnitPrice'])

# Display the cleaned dataframe
ecommerce_retail
```
| InvoiceNo | StockCode | Description                          | Quantity | InvoiceDate          | UnitPrice | CustomerID | Country        |
|-----------|-----------|--------------------------------------|----------|----------------------|-----------|------------|----------------|
| 536365    | 85123A    | WHITE HANGING HEART T-LIGHT HOLDER  | 6        | 2010-12-01 08:26:00 | 2.55      | 17850.0    | United Kingdom |
| 536365    | 71053     | WHITE METAL LANTERN                 | 6        | 2010-12-01 08:26:00 | 3.39      | 17850.0    | United Kingdom |
| 536365    | 84406B    | CREAM CUPID HEARTS COAT HANGER      | 8        | 2010-12-01 08:26:00 | 2.75      | 17850.0    | United Kingdom |
| 536365    | 84029G    | KNITTED UNION FLAG HOT WATER BOTTLE | 6        | 2010-12-01 08:26:00 | 3.39      | 17850.0    | United Kingdom |
| 536365    | 84029E    | RED WOOLLY HOTTIE WHITE HEART       | 6        | 2010-12-01 08:26:00 | 3.39      | 17850.0    | United Kingdom |
| ...       | ...       | ...                                  | ...      | ...                  | ...       | ...        | ...            |
| 581587    | 22613     | PACK OF 20 SPACEBOY NAPKINS         | 12       | 2011-12-09 12:50:00 | 0.85      | 12680.0    | France         |
| 581587    | 22899     | CHILDREN'S APRON DOLLY GIRL         | 6        | 2011-12-09 12:50:00 | 2.10      | 12680.0    | France         |
| 581587    | 23254     | CHILDRENS CUTLERY DOLLY GIRL        | 4        | 2011-12-09 12:50:00 | 4.15      | 12680.0    | France         |
| 581587    | 23255     | CHILDRENS CUTLERY CIRCUS PARADE     | 4        | 2011-12-09 12:50:00 | 4.15      | 12680.0    | France         |
| 581587    | 22138     | BAKING SET 9 PIECE RETROSPOT        | 3        | 2011-12-09 12:50:00 | 4.95      | 12680.0    | France         |
### RFM Calculation and Segmentation
```python
# Set a reference date for calculating recency (the latest date in the dataset)
reference_date = pd.to_datetime('2011-12-31')

# Calculate the total amount for each transaction by multiplying Quantity and UnitPrice
ecommerce_retail['TotalAmount'] = ecommerce_retail['Quantity'] * ecommerce_retail['UnitPrice']

# Create an RFM (Recency, Frequency, Monetary) dataframe by grouping data by CustomerID
rfm = ecommerce_retail.groupby('CustomerID').agg({
    # Recency: calculate the number of days since the customer's last purchase by finding the max InvoiceDate for each customer
    'InvoiceDate': lambda x: (reference_date - x.max()).days,
    # Frequency: count the unique invoices (number of purchases) for each customer
    'InvoiceNo': 'nunique',
    # Monetary: sum the total amount spent by each customer
    'TotalAmount': 'sum'
}).reset_index().rename(columns={
    'InvoiceDate': 'Recency',   # Rename InvoiceDate column to Recency
    'InvoiceNo': 'Frequency',   # Rename InvoiceNo column to Frequency
    'TotalAmount': 'Monetary'   # Rename TotalAmount column to Monetary
})

# Display the RFM table
rfm
```
| CustomerID | Recency | Frequency | Monetary |
|------------|---------|-----------|----------|
| 12346.0    | 346     | 1         | 77183.60 |
| 12347.0    | 23      | 7         | 4310.00  |
| 12348.0    | 96      | 4         | 1797.24  |
| 12349.0    | 39      | 1         | 1757.55  |
| 12350.0    | 331     | 1         | 334.40   |
| ...        | ...     | ...       | ...      |
| 18280.0    | 298     | 1         | 180.60   |
| 18281.0    | 201     | 1         | 80.82    |
| 18282.0    | 28      | 2         | 178.05   |
| 18283.0    | 24      | 16        | 2094.88  |
| 18287.0    | 63      | 3         | 1837.28  |

```python
# Create a Recency score (R_score) by dividing Recency values into 5 quantiles
# Higher scores indicate more recent purchases, with 5 being the most recent
rfm['R_score'] = pd.qcut(rfm['Recency'].rank(method='first', ascending=False), q=5, labels=[1,2,3,4,5])

# Create a Frequency score (F_score) by dividing Frequency values into 5 quantiles
# Higher scores indicate more frequent purchases, with 5 being the most frequent
rfm['F_score'] = pd.qcut(rfm['Frequency'].rank(method='first', ascending=True), q=5, labels=[1,2,3,4,5])

# Create a Monetary score (M_score) by dividing Monetary values into 5 quantiles
# Higher scores indicate higher monetary value, with 5 being the highest
rfm['M_score'] = pd.qcut(rfm['Monetary'].rank(method='first', ascending=True), q=5, labels=[1,2,3,4,5])

# Combine R_score, F_score, and M_score to create a single RFM_score for each customer
# This score is a concatenated string of the three individual scores
rfm['RFM_score'] = rfm['R_score'].astype(str) + rfm['F_score'].astype(str) + rfm['M_score'].astype(str)

# Display the RFM table with the new scores
rfm
```
| CustomerID | Recency | Frequency | Monetary | R_score | F_score | M_score | RFM_score |
|------------|---------|-----------|----------|---------|---------|---------|-----------|
| 12346.0    | 346     | 1         | 77183.60 | 1       | 1       | 5       | 115       |
| 12347.0    | 23      | 7         | 4310.00  | 5       | 5       | 5       | 555       |
| 12348.0    | 96      | 4         | 1797.24  | 2       | 4       | 4       | 244       |
| 12349.0    | 39      | 1         | 1757.55  | 4       | 1       | 4       | 414       |
| 12350.0    | 331     | 1         | 334.40   | 1       | 1       | 2       | 112       |
| ...        | ...     | ...       | ...      | ...     | ...     | ...     | ...       |
| 18280.0    | 298     | 1         | 180.60   | 1       | 2       | 1       | 121       |
| 18281.0    | 201     | 1         | 80.82    | 1       | 2       | 1       | 121       |
| 18282.0    | 28      | 2         | 178.05   | 5       | 3       | 1       | 531       |
| 18283.0    | 24      | 16        | 2094.88  | 5       | 5       | 5       | 555       |
| 18287.0    | 63      | 3         | 1837.28  | 3       | 4       | 4       | 344       |

```python
# Convert the 'RFM Score' column to string type to facilitate string operations
seg['RFM Score'] = seg['RFM Score'].astype(str)

# Split the 'RFM Score' strings into lists based on the comma delimiter
seg['RFM Score'] = seg['RFM Score'].str.split(',')

# Explode the lists in 'RFM Score' into separate rows while resetting the index
# This creates a new row for each score, allowing for individual RFM score analysis
seg = seg.explode('RFM Score').reset_index(drop=True).rename(columns={'RFM Score': 'RFM_score'})

# Display the segmented data with the new RFM_score column
seg
```
| Segment            | RFM_score |
|--------------------|-----------|
| Champions          | 555       |
| Champions          | 554       |
| Champions          | 544       |
| Champions          | 545       |
| Champions          | 454       |
| ...                | ...       |
| Lost customers     | 112       |
| Lost customers     | 121       |
| Lost customers     | 131       |
| Lost customers     | 141       |
| Lost customers     | 151       |

```python
# Remove any leading or trailing whitespace characters from the 'RFM_score' column in both rfm and seg DataFrames
rfm['RFM_score'] = rfm['RFM_score'].str.strip()
seg['RFM_score'] = seg['RFM_score'].str.strip()

# Merge the rfm and seg DataFrames on the 'RFM_score' column using a left join
# This will combine the RFM data with segment information based on matching RFM scores
joined = rfm.merge(seg, on='RFM_score', how='left')

# Display the merged DataFrame containing RFM scores along with their corresponding segments
joined
```
| CustomerID | Recency | Frequency | Monetary  | R_score | F_score | M_score | RFM_score | Segment          |
|------------|---------|-----------|-----------|---------|---------|---------|-----------|-------------------|
| 12346.0    | 346     | 1         | 77183.60  | 1       | 1       | 5       | 115       | Cannot Lose Them   |
| 12347.0    | 23      | 7         | 4310.00   | 5       | 5       | 5       | 555       | Champions          |
| 12348.0    | 96      | 4         | 1797.24   | 2       | 4       | 4       | 244       | At Risk            |
| 12349.0    | 39      | 1         | 1757.55   | 4       | 1       | 4       | 414       | Promising          |
| 12350.0    | 331     | 1         | 334.40    | 1       | 1       | 2       | 112       | Lost customers     |
| ...        | ...     | ...       | ...       | ...     | ...     | ...     | ...       | ...                |
| 18280.0    | 298     | 1         | 180.60    | 1       | 2       | 1       | 121       | Lost customers     |
| 18281.0    | 201     | 1         | 80.82     | 1       | 2       | 1       | 121       | Lost customers     |
| 18282.0    | 28      | 2         | 178.05    | 5       | 3       | 1       | 531       | Potential Loyalist |
| 18283.0    | 24      | 16        | 2094.88   | 5       | 5       | 5       | 555       | Champions          |
| 18287.0    | 63      | 3         | 1837.28   | 3       | 4       | 4       | 344       | Loyal              |

```python
# Group the joined DataFrame by 'Segment' and count the number of unique customers
grp = joined.groupby('Segment')['CustomerID'].nunique().reset_index().rename(columns={'CustomerID': 'Cust_count'})

# Calculate the share of each segment's customer count relative to the total unique customers
grp['Count_share'] = grp['Cust_count'] / joined['CustomerID'].nunique()

# Group by 'Segment' and sum the 'Monetary' values to get total revenue per segment
Revenue = joined.groupby('Segment')['Monetary'].sum()

# Merge the revenue data back into the grouped DataFrame
grp = grp.merge(Revenue, on='Segment').rename(columns={'Monetary': 'Revenue'})

# Calculate the share of each segment's revenue relative to the total revenue
grp['Revenue_share'] = grp['Revenue'] / joined['Monetary'].sum()

# Display the grouped DataFrame with segments, customer counts, and revenue shares
grp
```
| Segment                   | Cust_count | Count_share | Revenue      | Revenue_share |
|---------------------------|------------|-------------|--------------|---------------|
| About To Sleep            | 282        | 0.065007    | 78234.770    | 0.008779      |
| At Risk                   | 424        | 0.097741    | 753236.571   | 0.084525      |
| Cannot Lose Them          | 92         | 0.021208    | 205324.720   | 0.023041      |
| Champions                 | 830        | 0.191332    | 5582918.870  | 0.626491      |
| Hibernating customers     | 697        | 0.160673    | 286069.592   | 0.032102      |
| Lost customers            | 491        | 0.113186    | 98376.750    | 0.011039      |
| Loyal                     | 433        | 0.099816    | 1042682.200  | 0.117005      |
| Need Attention            | 277        | 0.063854    | 459753.971   | 0.051592      |
| New Customers             | 263        | 0.060627    | 58782.530    | 0.006596      |
| Potential Loyalist        | 414        | 0.095436    | 225740.110   | 0.025332      |
| Promising                 | 135        | 0.031120    | 120287.820   | 0.013498      |
### Visualization
```python
# Define a list of column names to analyze the distribution of.
colnames = ['Recency', 'Frequency', 'Monetary']

# Loop through each column name in the colnames list.
for col in colnames:
    # Create a new figure and axis for the plot with specified size (12 inches wide and 3 inches tall).
    fig, ax = plt.subplots(figsize=(12, 3))
    
    # Create a distribution plot for the current column using seaborn's distplot function.
    # This will show the probability density function of the data in that column.
    sns.distplot(joined[col])
    
    # Set the title of the plot to indicate which column's distribution is being displayed.
    ax.set_title('Distribution of %s' % col)
    
    # Display the plot.
    plt.show()
```
![image](https://github.com/user-attachments/assets/c49e0c25-d81f-4bb8-aea7-9403b7d0bcc4)
![image](https://github.com/user-attachments/assets/d99245f4-a009-4f26-8d0f-5f9f78e2881e)
![image](https://github.com/user-attachments/assets/2bc87bd6-df7a-4005-aaca-439ed4a9a90f)
```python
# Define a list of colors to be used for the segments in the treemap.
colors = ['#FF0000', '#00FFFF', '#FFFF00', '#A52A2A', '#800080', '#FF00CB', '#FFA500', '#FF00FF', '#736F6E']

# Create a new figure and axis for the plot with specified size (15 inches wide and 8 inches tall).
fig, ax = plt.subplots(1, figsize=(15, 8))

# Create a treemap using the squarify library to visualize customer count by RFM segments.
squarify.plot(sizes=grp['Cust_count'],  # The sizes of the squares are based on customer count for each segment.
              label=grp['Segment'],    # The labels for each segment in the treemap.
              value=[f'{x*100:.2f}%' for x in grp['Count_share']],  # Show percentage of total customers for each segment.
              alpha=.8,                # Set the transparency level of the squares.
              color=colors,            # Use the defined color list for the squares.
              bar_kwargs=dict(linewidth=1.5, edgecolor='white'))  # Set properties for the border of the squares.

# Set the title of the plot with a specified font size.
plt.title('RFM Segments of Customer Count', fontsize=16)

# Hide the axes for a cleaner look.
plt.axis('off')

# Display the plot.
plt.show()
```
![image](https://github.com/user-attachments/assets/a2208dc8-ff6d-4fdf-9d78-11ddb1217075)
```python
# Define a list of colors to be used for the segments in the treemap.
colors = ['#FF0000', '#00FFFF', '#FFFF00', '#A52A2A', '#800080', '#FF00CB', '#FFA500', '#FF00FF', '#736F6E']

# Create a new figure and axis for the plot with specified size (15 inches wide and 8 inches tall).
fig, ax = plt.subplots(1, figsize=(15, 8))

# Create a treemap using the squarify library to visualize revenue by RFM segments.
squarify.plot(sizes=grp['Revenue'],  # The sizes of the squares are based on the total revenue for each segment.
              label=grp['Segment'],  # The labels for each segment in the treemap.
              value=[f'{x*100:.2f}%' for x in grp['Revenue_share']],  # Show percentage of total revenue for each segment.
              alpha=.8,                # Set the transparency level of the squares.
              color=colors,            # Use the defined color list for the squares.
              bar_kwargs=dict(linewidth=1.5, edgecolor='white'))  # Set properties for the border of the squares.

# Set the title of the plot with a specified font size.
plt.title('RFM Segments of Customer Revenue', fontsize=16)

# Hide the axes for a cleaner look.
plt.axis('off')

# Display the plot.
plt.show()
```
![image](https://github.com/user-attachments/assets/1e1c4a0d-165d-414e-a9c7-9660ba79985c)
## Insight & Recommendatiom
### General recommendation 
![image](https://github.com/user-attachments/assets/778384e0-d99d-4915-959b-9da8c5f61cc8)
### Detailed recommendation for each segment 
![image](https://github.com/user-attachments/assets/bfe5753c-a86e-4f8f-a0de-c18a12767b1a)
In conclusion, the business is performing well in retaining the Champions group – the most important customer segment. However, there is a need to improve strategies for other segments to optimize revenue. Key segments to focus on include:
- Champions (19.13% of customers, 62.65% of revenue): This is the most loyal and highest value customer group.
- Loyal (9.98% of customers, 11.70% of revenue): This group also holds high value and has the potential to become Champions with proper care.
- At Risk (9.77% of customers, 8.45% of revenue): This group is at risk of leaving the business but still contributes significantly to revenue.
- Lost Customers (11.32% of customers, 1.10% of revenue): Although contributing little to revenue, this group represents a large percentage of total customers.
## Conclusion
In summary, effective customer segmentation is essential for optimizing marketing strategies and enhancing customer retention. By identifying and understanding the distinct characteristics of each segment, particularly the Champions, Loyal, At Risk, and Lost Customers, the business can tailor its approach to maximize engagement and revenue. Implementing targeted strategies for these key segments will not only strengthen customer loyalty but also drive sustainable growth in the long term.





