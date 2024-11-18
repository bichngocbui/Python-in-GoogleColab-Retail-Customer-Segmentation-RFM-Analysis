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

![image](https://github.com/user-attachments/assets/59f4a4b3-fa74-4598-ad26-a3b35e4ab12e)
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
![image](https://github.com/user-attachments/assets/015b2389-cba5-466b-b5c8-d704227ca292)

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
![image](https://github.com/user-attachments/assets/a603f396-ae35-45d7-87ea-dcbe2542a689)

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
![image](https://github.com/user-attachments/assets/f8335899-3405-4166-a5f4-170907a5eddf)

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
![image](https://github.com/user-attachments/assets/32e6578e-6a18-4479-9c15-5febf3bf020a)

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
![image](https://github.com/user-attachments/assets/da807937-019c-43f6-92ef-e98e1b22bb65)

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
![image](https://github.com/user-attachments/assets/c1816772-42ad-4745-b52d-fcaa6d108d2a)

### Visualization & Recommendation 
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
#### Insight
- The peak distribution from 20-25 days indicates that customers typically return to make purchases after 1-2 months.
- Most customers have transacted in the last 3-4 months.
- A small number of customers have not transacted for more than 6 months to over a year.
#### Action 
- Retain frequent customers: For customers who purchase every 1-2 months, maintain engagement through regular promotional programs and newsletters about new products.
- Reactivate low-frequency customers: For those who haven’t purchased in over 6 months, implement special campaigns such as personalized offers and exclusive promotions to encourage them to return.
- Closely monitor customers nearing their buying cycle: Establish channels to remind customers of new offers or products of interest as they approach 1-2 months without a purchase.
![image](https://github.com/user-attachments/assets/d99245f4-a009-4f26-8d0f-5f9f78e2881e)
#### Insight
- Strong right skew, with most customers making fewer than 50 purchases.
- The peak distribution is close to 0, indicating that most customers only purchase 1-2 times.
- A small number of customers have high purchase frequency (over 150 times).
#### Action 
- Increase purchase frequency for low-frequency customers: Utilize remarketing strategies or provide incentives for the next purchase immediately after the first transaction to encourage customers to return.
- Loyalty programs: For customers who have purchased more than 1-2 times, develop point accumulation or discounts for future purchases to increase shopping frequency.
- Maintain high-frequency customers: Create special programs or personalized service packages for customers with high purchase frequency to ensure their continued loyalty to the brand.
![image](https://github.com/user-attachments/assets/2bc87bd6-df7a-4005-aaca-439ed4a9a90f)
### Insight 
- Strong right skew, with most customers having low spending values.
- A small number of customers have extremely high spending values (up to 250,000 currency units).
#### Action 
- Upsell and cross-sell: For customers with low spending value, the business should enhance the introduction of related products (cross-sell) or higher-end versions of products they are purchasing (upsell).
- Increase average order value: Use strategies like "combo," "buy more save more," or "free shipping on orders above a certain amount" to encourage customers to spend more per order.
- Special care for high-value customers: For customers with high spending value, establish VIP offers, invite them to events, or offer experiences with new products to increase engagement and maintain high consumption levels.
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
#### Champion 
##### Insight
Customers frequently return to purchase within 1-2 months with very high frequency. They spend significantly, contributing greatly to revenue, and are the most loyal and valuable customers.
##### Recommendation
- Maintain engagement: Provide regular promotional programs and newsletters about new products
- Loyalty program: Reward points for the next transaction
- VIP care: Create VIP offers, invite to events, or new product experiences to retain high-value customers.
#### Loyal
##### Insight
Customers frequently return in the last 3-4 months, with a relatively high purchase frequency but not as high as Champions. They spend an average amount and have potential to become premium customers with proper care.
##### Recommendation
- Encourage through appreciation programs: Send promotions based on purchase history.
- Loyalty program: Apply discount policies for the next purchase to maintain purchasing frequency.
#### Potential Loyalist
##### Insight
Customers with the potential to become loyal, have purchased in the last 3-4 months, but their purchase frequency and value are still low. They show signs of buying more but are not stable yet.
##### Recommendation
- Care to become loyal customers: Provide suitable offers, product suggestions, and promotions to increase purchase frequency.
- Combo policy: Encourage more purchases with combo packs or discounts for bulk buying to increase order value.
#### Promising
##### Insight
Potential customers who have purchased recently but have low purchase frequency and value. They can develop into loyal customers with proper care.
##### Recommendation
- Build loyalty: Use remarketing campaigns with discount codes for their next purchase right after their first transaction.
- Encourage additional purchases: Increase purchase frequency with special offers for the next purchase or loyalty programs.
#### New Customers
##### Insight
New customers who typically purchase 1-2 times with low purchase value. They may return in the future if encouraged and cared for properly.
##### Recommendation
- Encourage the next purchase: Provide welcome offers or discounts for their next purchase to encourage them to return.
- Product introduction: Suggest similar or popular products to increase the chances of additional purchases.
#### Need Attention
##### Insight
Customers who used to purchase frequently but have decreased frequency in the last 3-6 months. They still spend considerably but show signs of decline and need attention to retain.
##### Recommendation
- Reactivate customers: Send personalized promotional programs or offers to capture their attention.
- Encourage return: Create remarketing campaigns or announce new products to entice them to continue shopping.
#### At Risk
##### Insight
Customers who have stopped purchasing for over 6 months. They previously had high purchase frequency and value but are now declining. This group is at high risk of switching to other brands without timely care.
##### Recommendation
- Special retention strategies: Provide personalized offers or significant discounts to encourage their return.
- Win-back program: Create special campaigns with attractive offers targeted at customers to return after a period of inactivity.
#### Hibernating Customers
##### Insight
Customers who have not purchased for a very long time, nearly ceasing transactions. They have low purchase value and contribute little to revenue.
##### Recommendation
- Reactivation through win-back campaigns: Send marketing campaigns with significant promotions, special offers, or incentives to encourage customers to return to shopping.
- Analyze reasons for loss: Understand why they stopped purchasing and adjust strategies accordingly.
#### Cannot Lose Them
##### Insight
A crucial group of customers with high purchase value, but currently have reduced purchasing frequency. This group needs priority care to avoid total loss.
##### Recommendation
- Retain with personalized programs: Provide special offers, VIP programs, or personalized care to ensure they return to shopping.
- Priority outreach: Create remarketing campaigns or product experience campaigns to capture their attention.
#### Lost Customers
##### Insight
Customers who have stopped transactions for a long time, indicating they are no longer interested in the products/services. These customers have very low purchase value and frequency. Losing these customers can negatively impact the company's revenue.
##### Recommendation
- Assess reasons for loss: Conduct surveys to understand why they did not return to shopping and their dissatisfaction points.
- Win-back program: Establish reactivation programs by offering attractive incentives, discounts, or free products for the first purchase upon return.
- Monitor effectiveness: Track reactivation campaigns to assess effectiveness and adjust as necessary.
#### About to Sleep
##### Insight
Customers showing signs of stopping shopping, with the recent return period extending beyond normal (nearly 6 months). They have low purchase frequency and value. This group is at risk of leaving the brand but can still recover if approached correctly.
##### Recommendation
- Quick reconnection: Send personalized offers to remind them of new products or special promotions.
- Encourage return: Create remarketing campaigns or provide significant offers to entice them to return to shopping before they transition to a higher risk group or are completely lost.
- Monitor: Track their behavior to implement appropriate retention strategies.
#### In conclusion, the business is performing well in retaining the Champions group – the most important customer segment. However, there is a need to improve strategies for other segments to optimize revenue. Key segments to focus on include:
- Champions (19.13% of customers, 62.65% of revenue): This is the most loyal and highest value customer group.
- Loyal (9.98% of customers, 11.70% of revenue): This group also holds high value and has the potential to become Champions with proper care.
- At Risk (9.77% of customers, 8.45% of revenue): This group is at risk of leaving the business but still contributes significantly to revenue.
- Lost Customers (11.32% of customers, 1.10% of revenue): Although contributing little to revenue, this group represents a large percentage of total customers.
## Conclusion
In summary, effective customer segmentation is essential for optimizing marketing strategies and enhancing customer retention. By identifying and understanding the distinct characteristics of each segment, particularly the Champions, Loyal, At Risk, and Lost Customers, the business can tailor its approach to maximize engagement and revenue. Implementing targeted strategies for these key segments will not only strengthen customer loyalty but also drive sustainable growth in the long term.





