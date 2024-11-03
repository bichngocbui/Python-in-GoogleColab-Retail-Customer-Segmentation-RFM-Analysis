# Python-RFM-Analysis-and-Customer-Segmentation-Visualization
## Project Overview
This project implements a comprehensive RFM (Recency, Frequency, Monetary) analysis system for SuperStore's global retail operations. The analysis aims to support the Marketing team's customer segmentation efforts for their year-end marketing campaigns, transitioning from manual Excel-based analysis to an automated Python solution due to increased data volume.
## Business Context
SuperStore, a global retail company, needs to segment their large customer base effectively for targeted Christmas and New Year marketing campaigns. The Marketing department requires a data-driven approach to:
- Identify and reward loyal customers
- Discover potential high-value customers
- Develop targeted marketing strategies for different customer segments
### Dataset Description
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


