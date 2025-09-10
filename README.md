# pusula_talent_academy
This repo was created for the solutions of the database case study questions given within the scope of Pusula Talent Academy 2025.
# Pusula Talent Academy 2025 - Database Case Study Solutions

## Question 1: Performance & Scalability Analysis in Hospital Data
### Scenario
A table has been used for 5 years in a Hospital Information Management System (HBYS).  
Each day, about 25,000 rows are inserted.  
Recently, queries on this table have become slower and users have reported difficulty accessing past records.

```sql
CREATE TABLE HastaIslemLog (
    Id INT IDENTITY(1,1) PRIMARY KEY,
    HastaId INT,
    IslemTarihi DATETIME,
    IslemKodu NVARCHAR(20),
    Aciklama NVARCHAR(500)
);
1. Reasons for performance degradation
Continuous insertion into the same table → Table size grows significantly over time.

The column Aciklama NVARCHAR(500) increases row size and query cost.

Lack of proper indexing → Queries result in full table scans.

2. Suggested improvements
Archiving: Move old records into archive tables (e.g., yearly partitions).

Indexing: Create indexes on frequently queried columns. Example:

sql
Kodu kopyala
CREATE NONCLUSTERED INDEX IX_HastaId ON HastaIslemLog (HastaId);
CREATE NONCLUSTERED INDEX IX_IslemTarihi ON HastaIslemLog (IslemTarihi);
This avoids full table scans and allows faster lookups.

3. Was using the table like this for 5 years correct?
Pros: Simple schema, easy to insert data.

Cons: Query performance decreases as data volume grows.

Conclusion: Long-term, a single table is not sustainable for heavy queries.

Question 2: Query Optimization
Scenario
The following query is frequently used by end users:

sql
Kodu kopyala
SELECT * 
FROM HastaKayit 
WHERE LOWER(AdSoyad) LIKE '%ahmet%' 
  AND YEAR(KayitTarihi) = 2024;
1. Problems with the query
YEAR(KayitTarihi) prevents index usage.

LOWER(AdSoyad) prevents index usage.

LIKE '%ahmet%' forces full scan.

SELECT * loads unnecessary columns.

2. Optimized query
sql
Kodu kopyala
SELECT AdSoyad, KayitTarihi 
FROM HastaKayit
WHERE AdSoyad LIKE 'ahmet%' 
  AND KayitTarihi >= '2024-01-01'
  AND KayitTarihi < '2025-01-01';
Replace YEAR() with a date range.

Avoid LOWER() → Use computed column with index if case-insensitive search is needed.

Avoid %...% when possible, prefer prefix search.

3. Application-side improvements
Implement autocomplete or search suggestions to reduce %...% queries.

Fetch only needed columns instead of SELECT *.

Question 3: T-SQL Query Challenge (Hospital Sales Example)
Scenario
The hospital pharmacy sells products. Sales and product details are stored in the following tables:

sql
Kodu kopyala
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
Sample Data:

sql
Kodu kopyala
-- Urun
(1, 'Laptop', 15000.00), 
(2, 'Mouse', 250.00), 
(3, 'Klavye', 450.00)

-- Satis
(1, 1, 2, '2024-01-10'),
(2, 2, 5, '2024-01-15'),
(3, 1, 1, '2024-02-20'),
(4, 3, 3, '2024-03-05'),
(5, 2, 7, '2024-03-25'),
(6, 3, 2, '2024-04-12')
Task 1: Total sales amount and quantity per year & product
sql
Kodu kopyala
SELECT 
    YEAR(S.SatisTarihi) AS SalesYear,
    U.UrunAdi,
    SUM(S.Adet * U.Fiyat) AS TotalSalesAmount,
    SUM(S.Adet) AS TotalQuantity
FROM Satis S
JOIN Urun U ON S.UrunID = U.UrunID
GROUP BY YEAR(S.SatisTarihi), U.UrunAdi
ORDER BY TotalSalesAmount DESC;
Task 2: Product with highest sales amount per year
sql
Kodu kopyala
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
);
Task 3: Products never sold
sql
Kodu kopyala
SELECT U.UrunID, U.UrunAdi
FROM Urun U
LEFT JOIN Satis S ON U.UrunID = S.UrunID
WHERE S.UrunID IS NULL;
