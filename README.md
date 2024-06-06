USE "database name"
DROP TABLE IF EXISTS Dim_Date
DROP TABLE IF EXISTS Dim_Product
DROP TABLE IF EXISTS Dim_TrancheAge
DROP TABLE IF EXISTS Fact_Sales
DROP TABLE IF EXISTS Fact_Monthly_Sales
GO
--creation de la dimmension date
CREATE TABLE [dbo].[Dim_Date]
(NoDate int identity(1,1) primary key,
Date datetime NOT NULL,
NoMois smallint NOT NULL,
NomMois varchar(10) NOT NULL,
Trimestre varchar(10) NOT NULL,
Annee smallint NOT NULL,
AnneeFiscal smallint NOT NULL,
MoisAnnee varchar(10) NOT NULL,
TrimestreAnnee varchar(10) NOT NULL,
TrimestreAnneeFiscal varchar(10) NOT NULL,
SemaineFiscal varchar(10) NOT NULL,
AnneeFiscalSemaineFiscal varchar(10) NOT NULL)
GO
select* from Dim_Date;
go 

--Insert dans la dimmension date

INSERT INTO [dbo].Dim_Date

SELECT  
--Date
P.[OrderDate] as date ,

--NoMois
month(P.[OrderDate]) as NoMois, 

--NomMois
DATENAME( MONTH, P.[OrderDate]) as NomMois,

--trimestre
CASE
	WHEN DATEPART(MONTH, P.[OrderDate]) IN (1,2,3)    THEN 'Q1'
	WHEN DATEPART(MONTH, P.[OrderDate]) IN (4,5,6)    THEN  'Q2'
	WHEN DATEPART(MONTH, P.[OrderDate]) IN (7,8,9)    THEN  'Q3'
	WHEN DATEPART(MONTH, P.[OrderDate]) IN (10,11,12) THEN  'Q4'
		END As Trimestre ,

--Annee
year(P.[OrderDate]) as years,

--AnneeFisclae
YEAR(DATEADD(MONTH, 4, P.OrderDate)) AS FiscalYear,

--MoisAnne
CONCAT (datepart (yy,P.[OrderDate]),+'-'+ FORMAT (P.[OrderDate],'MM')) as anneMois ,

--TrimestreAnnee
CASE
	WHEN DATEPART(MONTH, P.[OrderDate]) IN (1,2,3)    THEN CONCAT('Q1-' , year(P.[OrderDate]))
	WHEN DATEPART(MONTH, P.[OrderDate]) IN (4,5,6)    THEN CONCAT('Q2-' , year(P.[OrderDate]))
	WHEN DATEPART(MONTH, P.[OrderDate]) IN (7,8,9)    THEN  CONCAT('Q3-' , year(P.[OrderDate]))
	WHEN DATEPART(MONTH, P.[OrderDate]) IN (10,11,12) THEN  CONCAT('Q4-' , year(P.[OrderDate]))
		END As TrimestreAnne ,

--trimestreAnneeFiscale
CASE
	WHEN DATEPART(MONTH, P.[OrderDate]) IN (1,2,3)    THEN CONCAT('Q1-' , YEAR(DATEADD(MONTH, 6, P.OrderDate)))
	WHEN DATEPART(MONTH, P.[OrderDate]) IN (4,5,6)    THEN CONCAT('Q2-' , YEAR(DATEADD(MONTH, 6, P.OrderDate)))
	WHEN DATEPART(MONTH, P.[OrderDate]) IN (7,8,9)    THEN  CONCAT('Q3-' , YEAR(DATEADD(MONTH, 6, P.OrderDate)))
	WHEN DATEPART(MONTH, P.[OrderDate]) IN (10,11,12) THEN  CONCAT('Q4-' , YEAR(DATEADD(MONTH, 6, P.OrderDate)))
		END As TrimestreAnneFiscal,

--SemaineFiscale
CASE 
	WHEN (DATEPART(WEEK, P.[OrderDate]) - 4) <= 0  THEN DATEPART(WEEK, P.[OrderDate]) + 49
            ELSE DATEPART(WEEK, P.[OrderDate]) - 4
       END as SemaineFiscale,

--AnneeSemaineFiscal
CASE 
	WHEN MONTH(P.OrderDate) > 6 And (DATEPART(WEEK, P.OrderDate) - 4) <= 0
		then CONCAT( YEAR(P.OrderDate) + 1 ,'-' , DATEPART(WEEK, P.OrderDate) + 49)
	WHEN MONTH(P.OrderDate) > 6 And (DATEPART(WEEK, P.OrderDate) - 4) > 0
	 	then CONCAT( YEAR(P.OrderDate) + 1 ,'-' , DATEPART(WEEK, P.OrderDate) - 4)
	WHEN MONTH(P.OrderDate) <= 6 And (DATEPART(WEEK, P.OrderDate) - 4) <= 0
	 	then CONCAT( YEAR(P.OrderDate)  ,'-' , DATEPART(WEEK, P.OrderDate) + 49)
	WHEN MONTH(P.OrderDate) <= 6 And (DATEPART(WEEK, P.OrderDate) - 4) > 0
	 	then CONCAT( YEAR(P.OrderDate)  ,'-' , DATEPART(WEEK, P.OrderDate) - 4 )
			END AS AnneeSemaineFiscal 

From AdventureWorks2014.Purchasing.PurchaseOrderHeader P
go
select* from Dim_Date;
go 

--creation de la dimmension product
CREATE TABLE [dbo].[Dim_Product]
(NoProduit int identity(1,1) primary key,
NumeroProduit nvarchar(25) Not NULL,
NomProduit nvarchar(50) NOT NULL,
CouleurProduit nvarchar(10) NOT NULL,
coutStandardProduit money Not NULL,
PrixProduit money NOT NULL,
StyleProduit nvarchar(25) Not NULL,
NomSsCategProduit nvarchar(25) NOT NULL,
NomCategProduit nvarchar(25) NOT NULL)
GO
select* from Dim_Product;
go

--Insert dans la dimmension product
INSERT INTO Dim_Product 
SELECT  
 P.ProductNumber,P.Name as NomProduit,P.Color as CouleurProduit,P.StandardCost,P.ListPrice,P.Style, PS.Name, PC.Name			
from AdventureWorks2014.Production.Product P	
left join AdventureWorks2014.Production.ProductSubcategory PS on P.ProductSubcategoryID = PS.ProductSubcategoryID		
left join AdventureWorks2014.Production.ProductCategory PC on PS.ProductCategoryID = PC.ProductCategoryID
where P.Color IS NOT NULL AND P.Style IS NOT NULL
go 
select * from dbo.Dim_Product;
 go
 
--creation de la dimmension TrancheAge
CREATE TABLE [dbo].[Dim_TrancheAge]
(
NoTrancheAge int identity(1,1) primary key,
NomTrancheAge nvarchar(20) NOT NULL,
)
GO
select* from Dim_TrancheAge;
go

--Insert dans la dimmension trancheAge

INSERT INTO dbo.Dim_TrancheAge 
SELECT 
case 
when (year(POH.OrderDate)-year(HE.BirthDate)) between 1 and 24 then  '-25 ans'

when (year(POH.OrderDate)-year(HE.BirthDate)) between 25 AND 34 then '25 ans   34 ans'

when (year(POH.OrderDate)-year(HE.BirthDate)) between 35 AND 44 then '35 ans   44 ans'
else '45 ans et plus' 

END as TrancheAge
FROM AdventureWorks2014.Purchasing.PurchaseOrderHeader POH
left join AdventureWorks2014.HumanResources.Employee HE  on   POH.EmployeeID = HE.BusinessEntityID
go

select * from dbo.Dim_TrancheAge;
go


--creation fact-Sales
CREATE TABLE [dbo].[Fact_Sales]
(NoSales int identity(1,1) primary key,
NoProduit int Not NULL Foreign Key REFERENCES Dim_product(NoProduit),
NoDate int Not NULL Foreign Key REFERENCES Dim_Date(NoDate),
NoTrancheAge int Not NULL Foreign Key REFERENCES Dim_TrancheAge(NoTrancheAge),
Quantite_vendue int Not NULL,
MontantTotal money Not NULL,
);
go
select* from fact_sales;
go

--Insert dans fact-sales 
 INSERT INTO  [dbo].[Fact_Sales] (Quantite_vendue, MontantTotal, NoDate, NoProduit, NoTrancheAge)
SELECT
sum(aa.orderQty) Quantite_vendue,
sum(aa.lineTotal) Montant_Total,
bb.NoDate,
cc.NoProduit,
dd.NoTrancheAge

from AdventureWorks2014.Sales.SalesOrderDetail aa,
(
	SELECT
	POD.ProductID,
	POH.orderdate,
	d.NoDate

	from
	AdventureWorks2014.Purchasing.PurchaseOrderHeader POH,
	AdventureWorks2014.Purchasing.PurchaseOrderDetail POD,
	Dim_Date d

	where POH.PurchaseOrderID = POD.PurchaseOrderID and POH.OrderDate = d.date
) bb,
(
	SELECT
	pp.ProductID,
	p.NoProduit

	from
	Dim_Product p,
	AdventureWorks2014.Production.Product pp

	where pp.productNumber COLLATE DATABASE_DEFAULT = p.numeroProduit COLLATE DATABASE_DEFAULT
) cc,
Dim_TrancheAge dd

where aa.ProductID = bb.ProductID and cc.ProductID = aa.ProductID and dd.NoTrancheAge = bb.NoDate

GROUP BY aa.ProductID, bb.NoDate, cc.NoProduit, dd.NoTrancheAge
go
select* from fact_sales;
go

--creation fact_monthly_sales
 
Create table fact_monthly_sales(
NoMonthlySales int identity(1, 1) primary key,
NoProduit int not null Foreign Key References Dim_Product(NoProduit),
NoDate int Not NULL Foreign Key REFERENCES Dim_Date(NoDate),
Quantite_Vendue int not null,
NbreCommande int not null,
MontantTotal money not null);
go
select* from fact_monthly_sales;
go 
--Insert dans fact_monthly_sales  
INSERT INTO fact_monthly_sales (Quantite_Vendue, MontantTotal ,NbreCommande,NoDate, NoProduit)
SELECT
sum(aa.orderQty) Quantite_vendue,
sum(aa.lineTotal) Montant_total,
count(cc.NoProduit) NbreCommande,
bb.NoDate,
cc.NoProduit

from AdventureWorks2014.Sales.SalesOrderDetail aa,
(
	SELECT
	pod.ProductID,
	poh.orderdate,
	d.NoDate

	from
	AdventureWorks2014.Purchasing.PurchaseOrderHeader poh,
	AdventureWorks2014.Purchasing.PurchaseOrderDetail pod,
	Dim_Date d

	where poh.PurchaseOrderID = pod.PurchaseOrderID and poh.OrderDate = d.date
) bb,
(
	SELECT
	pp.ProductID,
	p.NoProduit

	FROM
	[Dim_Product] p,
	AdventureWorks2014.Production.Product pp

	where pp.productNumber COLLATE DATABASE_DEFAULT = p.numeroProduit COLLATE DATABASE_DEFAULT
) cc

where aa.ProductID = bb.ProductID and cc.ProductID = aa.ProductID

GROUP BY aa.ProductID, cc.NoProduit, YEAR(bb.OrderDate), MONTH(bb.OrderDate), bb.NoDate
GO

select*from fact_monthly_sales;
go