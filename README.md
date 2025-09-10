# pusula_talent_academy
This repo was created for the solutions of the database case study questions given within the scope of Pusula Talent Academy 2025.
--------------------------------------------------------------------------------------------------------------
#Question 1: Performance & Scalability Analysis in Hospital Data
Scenario:
A table below has been used for 5 years in a Hospital Information Management System (HBYS). Each day, about
25,000 rows are inserted.
Recently, queries on this table have become slower and users have reported difficulty accessing past records.
```sql
CREATE TABLE HastaIslemLog (
 Id INT IDENTITY(1,1) PRIMARY KEY,
 HastaId INT,
 IslemTarihi DATETIME,
 IslemKodu NVARCHAR(20),
 Aciklama NVARCHAR(500)
);
```
##1. What could be the reasons for the performance degradation?
###a) Continuous insertion into the same table  
Since data has been inserted into the same table every day for 5 years,  
the data volume has grown significantly, causing performance issues.  

###b) The "Aciklama" column is defined as NVARCHAR(500).  
This wide column makes queries heavier, especially since there is no indexing.  
If no filtering is applied, the query engine has to scan all rows and read long texts,  
which slows down performance.

##2. What improvements would you suggest for better sustainability?
###a) Create archive tables based on "IslemTarihi"  
For example, move records before 2023 into another table and delete them from the main table.  
This will reduce the data volume in `HastaIslemLog` and improve query performance.  

###b) Implement indexing  
Why indexing is needed can be explained simply:  
For example:

SELECT * FROM HastaIslemLog WHERE HastaId = 864;
Without an index, the query engine scans the entire table to find the matching row,
whether it is in the 10th row or the 1,000,000th row.

With an index:
CREATE NONCLUSTERED INDEX IX_HastaId ON HastaIslemLog (HastaId);
SQL Server keeps track of the sorted values and their positions.
Instead of scanning the whole table, it can jump directly to the required rows.
This improves read performance significantly.
Indexes can also be added on IslemTarihi or IslemKodu, depending on common queries.
For example, queries like "all operations in 2024" or "all patients with MR scans"
will run much faster with indexes.

However, indexes also bring maintenance overhead:
every insert will update both the table and its indexes.
Therefore, indexes should be chosen carefully based on frequently used queries.

##3.Do you think using the table in this way for 5 years was the correct approach? Why or why not?
Storing all the data in a single table over years makes query writing simpler.
However, as the data volume grows, queries become slower and harder to execute.
For example, searching for a specific patient record becomes much harder
when the table is very large.
---------------------------------------------------------------------------------------------------
Scenario:
The following query is frequently used by end users
```sql
SELECT * 
FROM HastaKayit 
WHERE LOWER(AdSoyad) LIKE '%ahmet%' AND YEAR(KayitTarihi) = 2024
```
#Question:2.What performance problems might arise from this query?
The use of the YEAR() function prevents index usage on the KayitTarihi column.
The use of LOWER() function also prevents index usage on the AdSoyad column.
LIKE '%ahmet%' with a leading wildcard forces a full scan instead of index seek.

SELECT * loads all columns unnecessarily, increasing I/O.

##1.How would you optimize this query and/or the table structure?
###a) Replace the YEAR() function with a date range filter:
```sql
SELECT * 
FROM HastaKayit
WHERE LOWER(AdSoyad) LIKE '%ahmet%' 
  AND KayitTarihi >= '2024-01-01'
  AND KayitTarihi < '2025-01-01';
```
This allows the database to use indexes on KayitTarihi.

###b) Similar to YEAR(), the LOWER() function reduces performance.
A persisted computed column (e.g., AdSoyadLower) could be added with an index
to improve performance for case-insensitive searches.

3.Are there any improvements that could be made on the application side?
Implement search suggestions or autocomplete to avoid %...% queries.
Use prefix searches (LIKE 'ahmet%') when possible.

--------------------------------------------------------------------------------------------------------

# Question 3: T-SQL Query Challenge (Hospital Sales Example)
 Scenario:
Pusula Talent Academy 2025 - SQL & DBA Case Study
 In the HBYS system, the hospital pharmacy sells products. Sales and product details are stored in the following tables:
```sql
 CREATE TABLE Urun (
    UrunID INT PRIMARY KEY,
    UrunAdi NVARCHAR(100),
    Fiyat DECIMAL(10,2)
 );
 CREATE TABLE Satis (
    SatisID INT PRIMARY KEY,
    UrunID INT FOREIGN KEY REFERENCES Urun(UrunID),
    Adet INT,
    SatisTarihi DATETIME
 );
 Sample Data:-- Urun
 (1, 'Laptop', 15000.00), (2, 'Mouse', 250.00), (3, 'Klavye', 450.00)-- Satis
 (1, 1, 2, '2024-01-10'), (2, 2, 5, '2024-01-15'), (3, 1, 1, '2024-02-20'),
 (4, 3, 3, '2024-03-05'), (5, 2, 7, '2024-03-25'), (6, 3, 2, '2024-04-12')
 Tasks:
## 1. Write a query that returns, per year and per product, the total sales amount (Fiyat * Adet) and total quantity.
    
SELECT 
    YEAR(S.SatisTarihi) AS SalesYear,
    U.UrunAdi,
    SUM(S.Adet * U.Fiyat) AS TotalSalesAmount,
    SUM(S.Adet) AS TotalQuantity
FROM Satis s
JOIN Urun u ON s.UrunID = u.UrunID
GROUP BY YEAR(s.SatisTarihi), u.UrunAdi
ORDER BY TotalSalesAmount DESC
        
 ##2. For each year, identify the product with the highest sales amount.
 
WITH YearlySales AS (
    SELECT 
        YEAR(S.SatisTarihi) AS SalesYear,
        U.UrunAdi,
        SUM(S.Adet * U.Fiyat) AS TotalSalesAmount
    FROM Satis S
    JOIN Urun U ON S.UrunID = U.UrunID
    GROUP BY YEAR(S.SatisTarihi), U.UrunAdi
) 
SELECT ys.SalesYear, ys.UrunAdi, ys.TotalSalesAmount
FROM YearlySales ys
WHERE ys.TotalSalesAmount = (
    SELECT MAX(TotalSalesAmount)
    FROM YearlySales y2
    WHERE y2.SalesYear = ys.SalesYear
)
    
 ##3. Write a query to list products that were never sold
  SELECT U.UrunID, U.UrunAdi
FROM Urun U
LEFT JOIN Satis S ON U.UrunID = S.UrunID
WHERE S.UrunID IS NULL
```
---------------------------------------------------------------------------------------------------
