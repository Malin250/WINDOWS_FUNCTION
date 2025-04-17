SQL Window Function Assignment

NAMES: MUGISHA Alain
ID: 25805

🎯 Objective
This project demonstrates the use of **SQL Window Functions** on a **sales dataset**, applying key functions such as:  
- `LAG()` , `LEAD()` → Comparing previous/next records  
- `RANK()` , `DENSE_RANK()` , `ROW_NUMBER()` → Ordering & ranking data  
- Aggregate functions → `MAX()` with and without `PARTITION BY`

 📊 Dataset Used
CREATE TABLE Sales (
    Transaction_ID INT PRIMARY KEY,
    Product_Name VARCHAR(50),
    Category VARCHAR(50),
    Customer_Name VARCHAR(50),
    Region VARCHAR(50),
    Sale_Date DATE,
    Quantity INT,
    Price_per_Unit DECIMAL(10,2)
);

INSERT INTO Sales VALUES (101, 'Widget A', 'Electronics', 'Alice Smith', 'East', DATE '2025-01-12', 5, 25.00);
INSERT INTO Sales VALUES (102, 'Widget B', 'Electronics', 'John Doe', 'West', DATE '2025-01-14', 2, 40.00);
INSERT INTO Sales VALUES (103, 'Widget A', 'Electronics', 'Jane Doe', 'East', DATE '2025-01-15', 3, 25.00);
INSERT INTO Sales VALUES (104, 'Widget C', 'Clothing', 'Bob Brown', 'South', DATE '2025-02-10', 4, 15.00);
INSERT INTO Sales VALUES (105, 'Widget B', 'Electronics', 'Alice Smith', 'East', DATE '2025-02-12', 1, 40.00);

//this is a sample dataset.
![dataset](https://github.com/user-attachments/assets/03a3b862-3cf9-4886-816b-0160b224be28)

1️⃣ LAG AND LEAD FUNCTION
SELECT 
    Transaction_ID,
    Product_Name,
    Sale_Date,
    Quantity,
    LAG(Quantity) OVER (ORDER BY Sale_Date) AS Previous_Quantity,
    LEAD(Quantity) OVER (ORDER BY Sale_Date) AS Next_Quantity,
    CASE
        WHEN Quantity > LAG(Quantity) OVER (ORDER BY Sale_Date)
             THEN 'HIGHER'
        WHEN Quantity < LAG(Quantity) OVER (ORDER BY Sale_Date)
             THEN 'LOWER'
        ELSE 'EQUAL'
    END AS Comparison_to_Previous
FROM Sales;
//this query is to compare values with previous or next records
//LAG() gets the previous while LEAD() gets the next record.
//CASE compares the current quantity with the previous one and labels it accordingly.
//this type of comparison help a business to track trends in product demand, aiding inventory adjustments.
![compare](https://github.com/user-attachments/assets/7237e496-7663-4f52-b572-c42fa13dfe25)

2️⃣ RANK & DENSE_RANK
SELECT 
    Category,
    Product_Name,
    SUM(Quantity) AS Total_Quantity,
    RANK() OVER (PARTITION BY Category ORDER BY SUM(Quantity) DESC) AS Rank,
    DENSE_RANK() OVER (PARTITION BY Category ORDER BY SUM(Quantity) DESC) AS Dense_Rank
FROM Sales
GROUP BY Category, Product_Name;
//RANK() leaves gap if there is ties values
//DENSE_RANK() does not produce gaps between tied rankings
//This can be used by sales teams or inventory managers to determine which products are best perfomers in each category.
![rank](https://github.com/user-attachments/assets/b9d107f7-7041-4c8e-ba42-2e3c8d4b00ea)

3️⃣ Fetching Top 3 Sales per Department
SELECT *
FROM (
    SELECT 
        Region,
        Product_Name,
        SUM(Quantity * Price_per_Unit) AS Total_Sales,
        RANK() OVER (PARTITION BY Region ORDER BY SUM(Quantity * Price_per_Unit) DESC) AS Rank
    FROM Sales
    GROUP BY Region, Product_Name
) RankedSales
WHERE Rank <= 3;
//Inner query aggregates sales by Producy_name and region and applies a rank() function per region based on total sales.
//outer query filters only the top 3 products (per region )
//This can be used to identify highest generating revenue products.
![region rank](https://github.com/user-attachments/assets/fb5eeb55-12be-4364-90c5-8540a37c0700)

4️⃣ ROW_NUMBER() Function
SELECT * FROM (
    SELECT 
        Category,
        Transaction_ID,
        Product_Name,
        Sale_Date,
        ROW_NUMBER() OVER (PARTITION BY Category ORDER BY Sale_Date ASC) AS Row_Num
    FROM Sales
) NumberedSales
WHERE Row_Num <= 2;
//ROW_NUMBER() assigns sequential numbers to each record within a category orderes by SALE_DATE
//the puter WHERE clause selects only the first two occurences meaning transactions per category
//This analysis can help HR to identify early trends or the first movers in a market segement, valuable for understanding customer behaviour in a new product.
![Row nmb](https://github.com/user-attachments/assets/183d917b-f36d-4893-9080-48bfdc6069f2)

5️⃣ AGGREGATION
SELECT 
    Category,
    Product_Name,
    Quantity,
    MAX(Quantity) OVER (PARTITION BY Category) AS Max_in_Category,
    MAX(Quantity) OVER () AS Max_Overall
FROM Sales;
//MAX(QUANTITY) OVER (PARTITION BY CATEGORY) computes the highest Quantity found in each category.
//MAX(Quantity) OVER () without a partition fetches the overall maximum across complete dataset.
//These insights help to spot highly successful transactions.
![aggregation](https://github.com/user-attachments/assets/b139a96d-488c-4499-95f5-d1eeb186b7de)


