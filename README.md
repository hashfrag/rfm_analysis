# rfm_analysis
RFM &amp; Customer Segmentation Analysis


![Dashboard 1](https://github.com/user-attachments/assets/1e8cef1a-fb64-45af-8aaf-d7bc70d8ef7f)


`code`   
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
      FROM rfm_values),

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

SELECT *, --Assigned group summary categories for additional filtering functionality
      CASE 
      WHEN rfm_segment IN ('Best Customers', 'Loyal Customers' , 'Big Spenders', 'New Customers' , 'High Potential Customers' , 'Low Potential Customers') THEN 'Active'
      WHEN rfm_segment IN ('Best Customers At Risk', 'Big Spenders At Risk', 'Loyal Customers At Risk' , 'High Potential Customers At Risk' , 'Low Potential Customers At Risk' ) THEN 'At Risk'
      WHEN rfm_segment IN ('Lost Low Potential Customers' , 'Lost High Potential Customers') THEN 'Lost'
      END AS rfm_group
FROM( SELECT *, -- Assigned segments for all RFM score combinations. Used individual scores instead of combined, for better detail when assigning them to segments.
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

      FROM  rfm_scores);

