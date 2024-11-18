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
## Bussiness vommendation
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





