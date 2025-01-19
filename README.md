# Customer Segmentation and RFM Modeling
![rfm_dashboard_screenshot](https://github.com/user-attachments/assets/3b0f1ef7-1a1e-4ed0-b418-e6a109ad3adc)
Link to the Tableau Dashboard: </br>
https://public.tableau.com/app/profile/ovidijus.pelakauskas/viz/M3S3RFM_17335051289480/Dashboard1 </br>

### Documentation <em>(click on individual topic to expand)</em>
<details><summary><strong>Goal of the Project</strong></summary><br>
Conduct a Customer Segmentation analysis and create a dashboard that would be helpful for future Customer Segmentation task. The users will be segmented based on 3 metrics - Recency, Frequency, and overall Monetary value for the business.
 
</details>

<details><summary><strong>Dataset Overview</strong></summary><br>
<em>Table Total Rows: <strong>541909</strong> </em><br>
  
| Row | InvoiceNo | StockCode | Description | Quantity | InvoiceDate | UnitPrice | CustomerID | Country |
| --- | --------- | --------- | ----------- | -------- | ----------- | --------- | ---------- | ------- |
|


</details>

<details><summary><strong>Technologies Used</strong></summary><br>
  Insert text here
</details>

<details><summary><strong>Data Preparation</strong></summary><br>
  Insert text here
</details>

<details><summary><strong>Data Visualization</strong></summary><br>
  Insert text here
</details>

<details><summary><strong>Analysis Results</strong></summary><br>
There are 4301 customers during the defined time period. 50% of those customers are still Active, 25% are currently At Risk of being Lost and the last 25% can be already considered as Lost. These customer groups are based on their last order recency. </br></br>
Active customers group is healthy. The average order recency is around 20 days, and frequency of orders is around 6 per user. Average revenue per user is also very high being at around 3000$. From all of these customers, only one segment needs more attention from us, those being the New Customers. They recently purchased something from us for the first time, and we should encourage them to come back.</br></br>

Customers that are currently At Risk have Average Order Recency of around 85 days. 3 Months from the last order is quite a long time, and some of those customers are quite valuable to the business, thus they need more attention from us, so that they would come back and shop with us again. </br></br>

<strong>Customer segments should be targeted in this order:</strong>
1. Best Customers and Big Spenders (At Risk). They result in the best monetary gains for the business, while having a smaller amount of people in those groups (lower amount of customers means less effort for the business while communicating with them). 
2. Loyal Customers (At Risk). While the monetary value acquired from them is not that great, the order frequency is very high. Plus there are only 41 customers in that segment. Keeping Loyal Customers is important for overall sustainability of the business.
3. High Potential Customers (At Risk). This is the biggest chunk of customers to try to get back, and there are 357 of them. They are the most average customers we have, but some of them have a lot of potential to be very good customers. 
4. New Customers. Not as important as the 4 segments above, but when those are dealt with, this would be the best next action.
* Low Potential Customers can be ignored until we do everything we can to get back the previously mentioned segments. Trying to get these customers will likely cost more to us than we will get in return from them.
</details>
