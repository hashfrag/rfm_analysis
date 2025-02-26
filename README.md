# Customer Segmentation and RFM Modeling
![rfm_dashboard_screenshot](https://github.com/user-attachments/assets/3b0f1ef7-1a1e-4ed0-b418-e6a109ad3adc)
Link to the Tableau Dashboard: </br>
https://public.tableau.com/app/profile/ovidijus.pelakauskas/viz/M3S3RFM_17335051289480/Dashboard1 </br>



## Goal of the Project
While working with the data from Online Commerce Store, conduct a Customer Segmentation analysis and create a dashboard that would be helpful for future Customer Segmentation tasks. The users will be segmented based on 3 metrics - Recency, Frequency, and overall Monetary value for the business.
<br><br>



## Technologies Used
<strong>Google BigQuery | </strong>For Exploratory Data Analysis, Data Cleaining, Manipulation, and Aggregation.<br>
<strong>Tableau | </strong>For Data Visualization and overall Dashboard creation.<br><br>



## Dataset Overview
<strong>Table Used for analysis:</strong>

| Row | InvoiceNo | StockCode | Description | Quantity | InvoiceDate | UnitPrice | CustomerID | Country |
| --- | --------- | --------- | ----------- | -------- | ----------- | --------- | ---------- | ------- |
|1|536414|22139|null|56|2010-12-01 11:52:00 UTC|0.0|null|United Kingdom|
|2|536544|22081|RIBBON REEL FLORA + FAUNA |1|2010-12-01 14:32:00 UTC|3.36|null|United Kingdom|
|3|536544|22100|SKULLS SQUARE TISSUE BOX|1|2010-12-01 14:32:00 UTC|2.51|null|United Kingdom|
|...  |...        |...        |...          |...       |...          |...        |...         |...      |
|541907|566405|22940|FELTCRAFT CHRISTMAS FAIRY|24|2011-09-12 13:41:00 UTC|4.25|17919|United Kingdom|
|541908|566405|23309|SET OF 60 I LOVE LONDON CAKE CASES|48|2011-09-12 13:41:00 UTC|0.55|17919|United Kingdom|
|541909|566405|22733|3D TRADITIONAL CHRISTMAS STICKERS|18|2011-09-12 13:41:00 UTC|1.25|17919|United Kingdom|

<em>Table Total Rows: <strong>541909</strong> </em><br>

<strong>| InvoiceNo   | </strong> Generated number for each order.<br>
<strong>| StockCode   | </strong> Purchased items code refferenced in internal systems (won't be needed for this analysis).<br>
<strong>| Description | </strong> Item description most of the time, sometimes used to indicate other processes when doing refunds, fixing accounting errors.<br>
<strong>| Quantity    | </strong> How many of each item was purchased.<br>
<strong>| InvoiceDate | </strong> Date when the order was placed.<br>
<strong>| UnitPrice   | </strong> Price of the item per 1 quantity.<br>
<strong>| CustomerID  | </strong> Unique Customer identifier.<br>
<strong>| Country     | </strong> Country in which the order was placed.<br>

Dataset has a lot of Null CustomerIDs, Negative Quantity Values (that were used to apply discounts or fix accounting errors), Null item descriptions, and all user order items were shown individually (same customerID could have 100 rows for one order).
<br><br>



## Working with Data

### Started off with initial data cleaning, aggregation, and Recency | Frequency | Monetary value calculations:
<em>
In WHERE clause I've set the time boundaries, filtered entries with no CustomerIds attached, and only showed entries where a positive amount of items were sold to avoid negative values. <br>
In GROUP BY clause I've grouped data to show combined R,F,M values for each individual user.<br>
In SELECT statement I've calculated:<br>
* Recency with Date Difference function by subtracting the latest order date of the customer from predetermined hypothetical current date).<br>
* Frequency by COUNTing all distinct InvoiceIDs each customer had.<br>
* Monetary value by adding up all purchases.</em>

<details><summary>Query <em>(dropdown)</em></summary>

    SELECT  
        CustomerID,
        Country,
        DATE_DIFF('2011-12-01' , CAST(MAX(InvoiceDate) AS DATE), DAY) AS recency,
        COUNT(DISTINCT InvoiceNO) AS frequency,
        SUM(UnitPrice*Quantity) AS monetary
    FROM  `tc-da-1.turing_data_analytics.rfm` AS original_table
    WHERE InvoiceDate BETWEEN '2010-12-01' AND '2011-11-30'
        AND CustomerID IS NOT NULL
        AND Quantity > 0 
        --Filtered out all of negative values for Monetary calculation that would result in negative numbers.
    GROUP BY CustomerID, Country
</details>


### With now acquired Recency, Frequency, and Monetary values for each customer, I've set quartile ranges that will be used for assigning scores later on:

<details><summary>Query <em>(dropdown)</em></summary>

    WITH  rfm_values AS (
        SELECT  
            CustomerID,
            Country,
            DATE_DIFF('2011-12-01' , CAST(MAX(InvoiceDate) AS DATE), DAY) AS recency,
            COUNT(DISTINCT InvoiceNO) AS frequency,
            SUM(UnitPrice*Quantity) AS monetary
        FROM  `tc-da-1.turing_data_analytics.rfm` AS original_table
        WHERE InvoiceDate BETWEEN '2010-12-01' AND '2011-11-30'
            AND CustomerID IS NOT NULL
            AND Quantity > 0 
            --Filtered out all of negative values for Monetary calculation that would result in negative numbers.
        GROUP BY CustomerID, Country )

    SELECT--Recency Quartiles Calculation
        APPROX_QUANTILES(Recency, 4)  [OFFSET(1)] AS r25,
        APPROX_QUANTILES(Recency, 4)  [OFFSET(2)] AS r50,
        APPROX_QUANTILES(Recency, 4)  [OFFSET(3)] AS r75,
        --Frequency Quartiles Calculation
        APPROX_QUANTILES(Frequency, 4)[OFFSET(1)] AS f25,
        APPROX_QUANTILES(Frequency, 4)[OFFSET(2)] AS f50,
        APPROX_QUANTILES(Frequency, 4)[OFFSET(3)] AS f75,
        --Monetary Quartiles Calculation
        APPROX_QUANTILES(Monetary, 4) [OFFSET(1)] AS m25,
        APPROX_QUANTILES(Monetary, 4) [OFFSET(2)] AS m50,
        APPROX_QUANTILES(Monetary, 4) [OFFSET(3)] AS m75
    FROM rfm_values
</details>


### Using the now set quartile ranges, I've assigned each customer their R,F,M scores from 1-4 (4 being the highest):
<details><summary>Query <em>(dropdown)</em></summary>

       WITH  rfm_values AS (
       SELECT  
             CustomerID,
             Country,
             DATE_DIFF('2011-12-01' , CAST(MAX(InvoiceDate) AS DATE), DAY) AS recency,
             COUNT(DISTINCT InvoiceNO) AS frequency,
             SUM(UnitPrice*Quantity) AS monetary
       FROM  `tc-da-1.turing_data_analytics.rfm` AS original_table
       WHERE InvoiceDate BETWEEN '2010-12-01' AND '2011-11-30'
             AND CustomerID IS NOT NULL
             AND Quantity > 0 
             --Filtered out all of negative values for Monetary calculation that would result in negative numbers.
       GROUP BY CustomerID, Country ),
 
       quartiles AS (
       SELECT--Recency Quartiles Calculation
             APPROX_QUANTILES(Recency, 4)  [OFFSET(1)] AS r25,
             APPROX_QUANTILES(Recency, 4)  [OFFSET(2)] AS r50,
             APPROX_QUANTILES(Recency, 4)  [OFFSET(3)] AS r75,
             --Frequency Quartiles Calculation
             APPROX_QUANTILES(Frequency, 4)[OFFSET(1)] AS f25,
             APPROX_QUANTILES(Frequency, 4)[OFFSET(2)] AS f50,
             APPROX_QUANTILES(Frequency, 4)[OFFSET(3)] AS f75,
             --Monetary Quartiles Calculation
             APPROX_QUANTILES(Monetary, 4) [OFFSET(1)] AS m25,
             APPROX_QUANTILES(Monetary, 4) [OFFSET(2)] AS m50,
             APPROX_QUANTILES(Monetary, 4) [OFFSET(3)] AS m75
       FROM rfm_values)


    SELECT 
        rfm_values.CustomerID,
        rfm_values.Country,
        rfm_values.Recency,
        rfm_values.Frequency,
        rfm_values.Monetary,
        -- Assigned Recency Quartiles
        CASE  WHEN rfm_values.Recency <= quartiles.r25 THEN 4
              WHEN rfm_values.Recency <= quartiles.r50 AND rfm_values.Recency > quartiles.r25 THEN 3
              WHEN rfm_values.Recency <= quartiles.r75 AND rfm_values.Recency > quartiles.r50 THEN 2
              ELSE 1 END AS R,
        -- Assigned Frequency Quartiles
        CASE  WHEN rfm_values.Frequency <= quartiles.f25 THEN 1
              WHEN rfm_values.Frequency <= quartiles.f50 AND rfm_values.Frequency > quartiles.f25 THEN 2
              WHEN rfm_values.Frequency <= quartiles.f75 AND rfm_values.Frequency > quartiles.f50 THEN 3
              ELSE 4 END AS F,
        -- Assigned Monetary Quartiles
        CASE  WHEN rfm_values.Monetary <= quartiles.m25 THEN 1
              WHEN rfm_values.Monetary <= quartiles.m50 AND rfm_values.Monetary > quartiles.m25 THEN 2
              WHEN rfm_values.Monetary <= quartiles.m75 AND rfm_values.Monetary > quartiles.m50 THEN 3
              ELSE 4 END AS M
      FROM rfm_values , quartiles
</details>


### Those scores were then used in final customers segmentation. 
<em>I’ve chosen to use RFM scores individualy for better detail instead of joining F and M scores together, because when trying to distinguish between loyal and big spending customers, combined FM score makes it not as accurate. <br>
The main segments are Best Customers, Loyal Customers, Big Spenders, High Potential and Low Potential Customers. The same ones apply for the At Risk category, but for the Lost segment I chose to show combined segments just for the High and Low Potential customers, as more in depth detail for that segment group seemed redundant. One outlier segment is in Active group, it being the New Customers segment.<br>
In the outer query I've added additional grouping for the segments, to be able to quickly identify Active, At Risk, and Lost customers.</em>

<details><summary>Query <em>(dropdown)</em></summary>

         rfm_scores AS (
         SELECT 
               rfm_values.CustomerID,
               rfm_values.Country,
               rfm_values.Recency,
               rfm_values.Frequency,
               rfm_values.Monetary,
               -- Assigned Recency Quartiles
               CASE  WHEN rfm_values.Recency <= quartiles.r25 THEN 4
                     WHEN rfm_values.Recency <= quartiles.r50 AND rfm_values.Recency > quartiles.r25 THEN 3
                     WHEN rfm_values.Recency <= quartiles.r75 AND rfm_values.Recency > quartiles.r50 THEN 2
                     ELSE 1 END AS R,
               -- Assigned Frequency Quartiles
               CASE  WHEN rfm_values.Frequency <= quartiles.f25 THEN 1
                     WHEN rfm_values.Frequency <= quartiles.f50 AND rfm_values.Frequency > quartiles.f25 THEN 2
                     WHEN rfm_values.Frequency <= quartiles.f75 AND rfm_values.Frequency > quartiles.f50 THEN 3
                     ELSE 4 END AS F,
               -- Assigned Monetary Quartiles
               CASE  WHEN rfm_values.Monetary <= quartiles.m25 THEN 1
                     WHEN rfm_values.Monetary <= quartiles.m50 AND rfm_values.Monetary > quartiles.m25 THEN 2
                     WHEN rfm_values.Monetary <= quartiles.m75 AND rfm_values.Monetary > quartiles.m50 THEN 3
                     ELSE 4 END AS M
         FROM rfm_values , quartiles)
   
    SELECT *,
          CASE 
          WHEN rfm_segment IN ('Best Customers', 'Loyal Customers' , 'Big Spenders', 'New Customers' , 'High Potential Customers' , 'Low Potential Customers') THEN 'Active'
          WHEN rfm_segment IN ('Best Customers At Risk', 'Big Spenders At Risk', 'Loyal Customers At Risk' , 'High Potential Customers At Risk' , 'Low Potential Customers At Risk' ) THEN 'At Risk'
          WHEN rfm_segment IN ('Lost Low Potential Customers' , 'Lost High Potential Customers') THEN 'Lost'
          END AS rfm_group
    FROM( SELECT *,
          CASE  WHEN  (R = 4 AND F = 4 AND M = 4) OR
                      (R = 3 AND F = 4 AND M = 4)
                      THEN 'Best Customers'
                WHEN  (R = 4 AND F = 4 AND M = 1) OR
                      (R = 4 AND F = 4 AND M = 2) OR
                      (R = 4 AND F = 4 AND M = 3) OR
                      (R = 3 AND F = 4 AND M = 1) OR
                      (R = 3 AND F = 4 AND M = 2) OR
                      (R = 3 AND F = 4 AND M = 3)
                      THEN 'Loyal Customers'
                WHEN  (R = 4 AND F = 1 AND M = 4) OR
                      (R = 4 AND F = 2 AND M = 4) OR
                      (R = 4 AND F = 3 AND M = 4) OR
                      (R = 3 AND F = 1 AND M = 4) OR
                      (R = 3 AND F = 2 AND M = 4) OR
                      (R = 3 AND F = 3 AND M = 4) 
                      THEN 'Big Spenders'
                WHEN  (R = 4 AND F = 1 AND M = 1) OR
                      (R = 4 AND F = 1 AND M = 2) OR
                      (R = 4 AND F = 1 AND M = 3)  
                      THEN 'New Customers'
                WHEN  (R = 4 AND F = 3 AND M = 1) OR
                      (R = 4 AND F = 3 AND M = 2) OR
                      (R = 4 AND F = 3 AND M = 3) OR
                      (R = 4 AND F = 2 AND M = 1) OR
                      (R = 4 AND F = 2 AND M = 2) OR
                      (R = 4 AND F = 2 AND M = 3) OR
                      (R = 3 AND F = 2 AND M = 3) OR
                      (R = 3 AND F = 3 AND M = 2) OR 
                      (R = 3 AND F = 3 AND M = 3) OR 
                      (R = 3 AND F = 1 AND M = 3) OR 
                      (R = 3 AND F = 3 AND M = 1)
                      THEN 'High Potential Customers'
                WHEN  (R = 3 AND F = 2 AND M = 2) OR
                      (R = 3 AND F = 2 AND M = 1) OR 
                      (R = 3 AND F = 1 AND M = 2) OR
                      (R = 3 AND F = 1 AND M = 1)           
                      THEN 'Low Potential Customers'
          -- R2
                WHEN  (R = 2 AND F = 4 AND M = 4) 
                      THEN 'Best Customers At Risk'
                WHEN  (R = 2 AND F = 1 AND M = 4) OR
                      (R = 2 AND F = 2 AND M = 4) OR
                      (R = 2 AND F = 3 AND M = 4) 
                      THEN 'Big Spenders At Risk'
                WHEN  (R = 2 AND F = 4 AND M = 1) OR
                      (R = 2 AND F = 4 AND M = 2) OR
                      (R = 2 AND F = 4 AND M = 3)     
                      THEN 'Loyal Customers At Risk' 
                WHEN  (R = 2 AND F = 2 AND M = 3) OR
                      (R = 2 AND F = 3 AND M = 2) OR 
                      (R = 2 AND F = 3 AND M = 3) OR 
                      (R = 2 AND F = 1 AND M = 3) OR 
                      (R = 2 AND F = 3 AND M = 1) 
                      THEN 'High Potential Customers At Risk' 
                WHEN  (R = 2 AND F = 2 AND M = 2) OR
                      (R = 2 AND F = 2 AND M = 1) OR 
                      (R = 2 AND F = 1 AND M = 2) OR
                      (R = 2 AND F = 1 AND M = 1)           
                      THEN 'Low Potential Customers At Risk'      
          -- R1
                WHEN  (R = 1 AND F = 4 AND M = 4) OR
                      (R = 1 AND F = 4 AND M = 3) OR 
                      (R = 1 AND F = 3 AND M = 4) OR
                      (R = 1 AND F = 3 AND M = 3) OR 
                      (R = 1 AND F = 4 AND M = 1) OR
                      (R = 1 AND F = 4 AND M = 2) OR 
                      (R = 1 AND F = 3 AND M = 1) OR
                      (R = 1 AND F = 3 AND M = 2) OR
                      (R = 1 AND F = 1 AND M = 4) OR
                      (R = 1 AND F = 1 AND M = 3) OR 
                      (R = 1 AND F = 2 AND M = 4) OR
                      (R = 1 AND F = 2 AND M = 3)       
                      THEN "Lost High Potential Customers"
                WHEN  (R = 1 AND F = 2 AND M = 2) OR 
                      (R = 1 AND F = 2 AND M = 1) OR
                      (R = 1 AND F = 1 AND M = 1) OR
                      (R = 1 AND F = 1 AND M = 2)
                      THEN 'Lost Low Potential Customers'    
                END AS rfm_segment 
          FROM  rfm_scores)
</details>



## Analysis Results
![Ov1WtBM](https://github.com/user-attachments/assets/1556533e-de65-4ae1-b3f0-7dfc53528e6d)


There are 4301 customers during the defined time period. 50% of those customers are still Active, 25% are currently At Risk of being Lost and the last 25% can be already considered as Lost. These customer groups are based on their last order recency. </br>
Active customers group is healthy. The average order recency is around 20 days, and frequency of orders is around 6 per user. Average revenue per user is also very high being at around 3000$. From all of these customers, only one segment needs more attention from us, those being the New Customers. They recently purchased something from us for the first time, and we should encourage them to come back.</br>

Customers that are currently At Risk have Average Order Recency of around 85 days. 3 Months from the last order is quite a long time, and some of those customers are quite valuable to the business, thus they need more attention from us, so that they would come back and shop with us again. </br>

<strong>Customer segments should be targeted in this order:</strong>
1. Best Customers and Big Spenders (At Risk). They result in the best monetary gains for the business, while having a smaller amount of people in those groups (lower amount of customers means less effort for the business while communicating with them). 
2. Loyal Customers (At Risk). While the monetary value acquired from them is not that great, the order frequency is very high. Plus there are only 41 customers in that segment. Keeping Loyal Customers is important for overall sustainability of the business.
3. High Potential Customers (At Risk). This is the biggest chunk of customers to try to get back, and there are 357 of them. They are the most average customers we have, but some of them have a lot of potential to be very good customers. 
4. New Customers. Not as important as the 4 segments above, but when those are dealt with, this would be the best next action.
* Low Potential Customers can be ignored until we do everything we can to get back the previously mentioned segments. Trying to get these customers will likely cost more to us than we will get in return from them.
</details>
