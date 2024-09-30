# SQL FMCG SALES PROJECT

In this project, I worked with sales data from an FMCG brand using SQL Server. The task involved applying advanced techniques such as join, union, subquery, update, case-when, and rank over to address the following questions:
- Use the newly created April and June sales table to find a list of customers who bought in April but did not buy in June and vice versa.

- Compare each customer's purchase amount in April and June.

- Calculate the sum of the June and April net gains for each store.

- Take out the total transaction amount of each store in 2019 and see what percentage it represents compared to the total transaction amount of each store in the sales table.

- Get the largest transaction amount of each employee with each store in 2019 and see what proportion of the total transaction of that employee in the store in 2019

- Calculate the total transaction amount of each employee with each store in each month of 2019 to see what proportion of the store's total revenue in each of those months of 2019.

- Update table Discount column MucChietKhau = 0.01 if MucToiThieu from 0 > 5, MucChietKhau = 0.02 if MucToiThieu from 6 > 10, MucChietKhau = 0.03 if MucToiThieu from 11 > 30, the remaining discount is 0.05

- Get the transaction information with the largest amount by day and by store

- Get the customer with the largest amount of money by month and by store

**Below are my SQL queries:**

_**1. Use the newly created April and June sales table to find a list of customers who bought in April but did not buy in June and vice versa.**_
```
SELECT 
	CUS_ID
	,Cus_name
FROM Portfolio.dbo.project
WHERE YEAR(trans_date) = 2019 
	AND MONTH(Trans_date) = 4
	AND cus_id NOT IN (
				SELECT cus_id
				FROM Portfolio.dbo.project
				WHERE YEAR(trans_date)= 2019 
				AND MONTH(trans_date)=6)
UNION
SELECT 
	CUS_ID
	,Cus_name
FROM Portfolio.dbo.project
WHERE YEAR(trans_date) = 2019 
	AND MONTH(Trans_date) = 6
	AND cus_id NOT IN (
				SELECT cus_id
				FROM Portfolio.dbo.project
				WHERE YEAR(trans_date)= 2019 
				AND MONTH(trans_date)=4)
```
![image](https://github.com/user-attachments/assets/f3dd24e0-72f9-4296-bcbb-ce396a9d9c10)

_**2. Compare each customer's purchase amount in April and June.**_

```
WITH cus_apr AS
(
    SELECT 
        cus_id
		, cus_name
		, SUM(LCY_AMT) AS Spent_apr
    FROM Portfolio.dbo.project
    WHERE YEAR(trans_date) = 2019 
      AND MONTH(trans_date) = 4
    GROUP BY cus_id, cus_name
),
cus_jun AS
(
    SELECT 
        cus_id,
        , cus_name
        , SUM(LCY_AMT) AS Spent_jun
    FROM Portfolio.dbo.project
    WHERE YEAR(trans_date) = 2019 
      AND MONTH(trans_date) = 6
    GROUP BY cus_id, cus_name
)

SELECT 
    COALESCE(cus_apr.cus_id, cus_jun.cus_id) AS cus_id
    , COALESCE(cus_apr.cus_name, cus_jun.cus_name) AS cus_name
    , ISNULL(Spent_apr, 0) AS Spent_apr
    , ISNULL(Spent_jun, 0) AS Spent_jun
    , ISNULL(Spent_apr, 0) - ISNULL(Spent_jun, 0) AS Diff
FROM cus_apr
FULL JOIN cus_jun
    ON cus_apr.cus_id = cus_jun.cus_id;
```
![image](https://github.com/user-attachments/assets/e3d757b4-512b-4b0d-b29a-42faf786a906)

_**3. Calculate the sum of the June and April net gains for each store.**_
```
SELECT
    ISNULL(A.storedid, B.storedid) AS storeid
    , ISNULL(A.Rev_apr, 0) AS rev_04
    , ISNULL(B.Rev_jun, 0) AS rev_06
    , ISNULL(B.Rev_jun, 0) - ISNULL(A.Rev_apr, 0) AS NET_06vs04
FROM
(
    SELECT 
        storedid
        ,SUM(LCY_AMT) AS Rev_apr
    FROM Portfolio.dbo.project
    WHERE YEAR(trans_date) = 2019 
      AND MONTH(trans_date) = 4
    GROUP BY storedid
) A
FULL JOIN
(
    SELECT 
        storedid
        , SUM(LCY_AMT) AS Rev_jun
    FROM Portfolio.dbo.project
    WHERE YEAR(trans_date) = 2019 
      AND MONTH(trans_date) = 6
    GROUP BY storedid
) B
ON A.storedid = B.storedid;
```
![image](https://github.com/user-attachments/assets/ff70f120-cfc3-4a7c-834f-906a5a8a9068)

_**4. Take out the total transaction amount of each store in 2019 and see what percentage it represents compared to the total transaction amount of each store in the sales table.**_
```
SELECT 
    ISNULL(A.STOREDID, B.STOREDID) AS STOREDID
    , ISNULL(A.rev_2019, 0) AS rev_2019
    , ISNULL(B.total_rev, 0) AS total_rev
    , CAST((A.rev_2019 / CAST(B.total_rev AS FLOAT)) * 100 AS DECIMAL(10,2)) AS [% rev_2019]
FROM
(
    SELECT 
        STOREDID
        , SUM(LCY_AMT) AS rev_2019
    FROM Portfolio.dbo.project
    WHERE YEAR(TRANS_DATE) = 2019
    GROUP BY STOREDID
) A
FULL JOIN 
(
    SELECT 
        STOREDID
        , SUM(LCY_AMT) AS total_rev
    FROM Portfolio.dbo.project
    GROUP BY STOREDID
) B
ON A.STOREDID = B.STOREDID;
```
![image](https://github.com/user-attachments/assets/02067dce-24de-4c81-8ded-14697c93bf71)

_**5.Calculate the total transaction amount of each employee with each store in each month of 2019 to see what proportion of the store's total revenue in each of those months of 2019.**_

```
WITH rev_per_sale AS
(
    SELECT
        STOREDID
        , SALE_ID
        , MONTH(TRANS_DATE) AS [MONTH]
        , SUM(LCY_AMT) AS Rev_sale
    FROM Portfolio.dbo.project
    WHERE YEAR(TRANS_DATE) = 2019
    GROUP BY
        STOREDID,
        SALE_ID,
        MONTH(TRANS_DATE)
),
rev_per_month AS
(
    SELECT 
        STOREDID
        , MONTH(TRANS_DATE) AS [MONTH]
        , SUM(LCY_AMT) AS total_month
    FROM Portfolio.dbo.project
    WHERE YEAR(TRANS_DATE) = 2019
    GROUP BY
        STOREDID,
        MONTH(TRANS_DATE)
)

SELECT
    ISNULL(A.STOREDID, B.STOREDID) AS STOREDID
    , A.SALE_ID
    , ISNULL(A.[MONTH], B.[MONTH]) AS [MONTH]
    , ISNULL(A.Rev_sale, 0) AS Rev_per_sale
    , ISNULL(B.total_month, 0) AS total_month
    , CAST((CAST(A.Rev_sale AS FLOAT) / CAST(B.total_month AS FLOAT)) * 100 AS DECIMAL(10, 2)) AS [% SALE_AMT]
FROM 
    rev_per_sale A
FULL JOIN 
    rev_per_month B
ON 
    A.STOREDID = B.STOREDID AND
    A.[MONTH] = B.[MONTH]
ORDER BY STOREDID, [MONTH];
```
![image](https://github.com/user-attachments/assets/8a9d70e8-c075-4c00-8247-e8008cc0d843)

_**6. Update table Discount column MucChietKhau = 0.01 if MucToiThieu from 0 > 5, MucChietKhau = 0.02 if MucToiThieu from 6 > 10, MucChietKhau = 0.03 if MucToiThieu from 11 > 30, the remaining discount is 0.05**_
```
UPDATE Portfolio.dbo.discount
SET MUC_CHIET_KHAU = 
	CASE
		WHEN MUC_TOI_THIEU BETWEEN 0 AND 5 THEN 0.01
		WHEN MUC_TOI_THIEU BETWEEN 6 AND 10 THEN 0.02
		WHEN MUC_TOI_THIEU BETWEEN 11 AND 30 THEN 0.03
		ELSE 0.05
	END
SELECT * FROM Portfolio.dbo.discount
```
![image](https://github.com/user-attachments/assets/08b22b71-a130-426d-ad36-c879bb523a4f)

_**7. Get the transaction information with the largest amount by day and by store**_
```
WITH Trans_day_store AS
(
SELECT 
	STOREDID
	, TRANS_DATE
	, CUS_ID
	, RETAIL_BILL
	, LCY_AMT
	, RANK () OVER (PARTITION BY STOREDID, TRANS_DATE ORDER BY LCY_AMT DESC) AS [rank]
FROM  Portfolio.dbo.project
)

SELECT *
FROM Trans_day_store
WHERE [rank] = 1
ORDER BY CAST(RIGHT(STOREDID,(LEN(STOREDID) - LEN('STORE '))) AS INT), TRANS_DATE
```
![image](https://github.com/user-attachments/assets/222a1f78-d501-41bd-ba13-94c07ecac97f)

_**8. Get the customer with the largest amount of money by month and by store**_
```
WITH Customer AS 
(
    SELECT 
        CUS_ID
        , CUS_NAME
        , STOREDID
        , MONTH(TRANS_DATE) AS [MONTH]
        , SUM(LCY_AMT) AS Spent
        , RANK() OVER (PARTITION BY STOREDID
		, MONTH(TRANS_DATE) ORDER BY SUM(LCY_AMT) DESC) AS [rank]
    FROM Portfolio.dbo.project
    GROUP BY
        STOREDID
        , MONTH(TRANS_DATE)
        , TRANS_DATE
        , CUS_ID
        , CUS_NAME
)
SELECT *
FROM Customer
WHERE [rank] = 1
ORDER BY CAST(RIGHT(STOREDID, (LEN(STOREDID) - LEN('STORE '))) AS INT), [MONTH]
```

![image](https://github.com/user-attachments/assets/c17cc585-8689-41df-a3ca-8f63519a3f2a)











