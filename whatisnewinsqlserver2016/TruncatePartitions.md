You can run this whole code in 1 shot 
```SQL

USE tempdb
GO

IF EXISTS(SELECT 1 FROM sys.tables WHERE name = 'SalesPartitioned')
DROP TABLE SalesPartitioned
GO

CREATE TABLE SalesPartitioned(
	YearCol SMALLINT NOT NULL,
	OrderID INT NOT NULL, 
	SomeData UNIQUEIDENTIFIER DEFAULT newsequentialid())
GO



INSERT SalesPartitioned (YearCol,OrderID)
SELECT 2013,number
FROM master..spt_values
WHERE type = 'P'
UNION ALL
SELECT 2014,number + 2048
FROM master..spt_values
WHERE type = 'P'
UNION ALL
SELECT 2015,number + 4096
FROM master..spt_values
WHERE type = 'P'
UNION ALL
SELECT 2016,number + 6144
FROM master..spt_values
WHERE type = 'P'
UNION ALL
SELECT 2017,number + 8192
FROM master..spt_values
WHERE type = 'P'
UNION ALL
SELECT 2018,number + 10240
FROM master..spt_values
WHERE type = 'P'



IF EXISTS(SELECT 1 FROM sys.partition_schemes WHERE name = 'psFiscalYear')
DROP PARTITION SCHEME psFiscalYear
GO

IF EXISTS(SELECT 1 FROM sys.partition_functions WHERE name = 'pfFiscalYear')
DROP PARTITION FUNCTION pfFiscalYear
GO

CREATE PARTITION FUNCTION pfFiscalYear(SMALLINT)
AS RANGE LEFT FOR VALUES(2013,2014,2015,2016,2017)
GO


CREATE PARTITION SCHEME psFiscalYear
AS PARTITION pfFiscalYear ALL TO ([PRIMARY])
GO

ALTER TABLE dbo.SalesPartitioned ADD CONSTRAINT
    PK_Sales PRIMARY KEY CLUSTERED (YearCol,OrderID)
ON psFiscalYear(YearCol)
GO

SELECT 'Partitions before running truncate statements' as Message

SELECT partition_number,rows 
FROM sys.partitions 
WHERE object_id = OBJECT_ID('SalesPartitioned')
 /*
partition_number	rows
1	2048
2	2048
3	2048
4	2048
5	2048
6	2048
*/
 
SELECT YearCol, $PARTITION.pfFiscalYear(YearCol) AS Partition, 
COUNT(*) AS [COUNT] FROM SalesPartitioned
GROUP BY $PARTITION.pfFiscalYear(YearCol),YearCol 
ORDER BY Partition;
GO
/*
YearCol	Partition	COUNT
2013	1	2048
2014	2	2048
2015	3	2048
2016	4	2048
2017	5	2048
2018	6	2048
*/


TRUNCATE TABLE SalesPartitioned 
WITH (PARTITIONS (2));  
GO  


SELECT 'Partitions after truncating partition 2' as Message

SELECT partition_number,rows 
FROM sys.partitions 
WHERE object_id = OBJECT_ID('SalesPartitioned')
/*
partition_number	rows
1	2048
2	0
3	2048
4	2048
5	2048
6	2048
*/
 
SELECT YearCol, $PARTITION.pfFiscalYear(YearCol) AS Partition, 
COUNT(*) AS [COUNT] FROM SalesPartitioned
GROUP BY $PARTITION.pfFiscalYear(YearCol),YearCol 
ORDER BY Partition;
GO

/*
YearCol	Partition	COUNT
2013	1	2048
2015	3	2048
2016	4	2048
2017	5	2048
2018	6	2048
*/



TRUNCATE TABLE SalesPartitioned 
WITH (PARTITIONS (4 TO 6));  
GO  

SELECT 'Partitions after truncating partition 4 to 6' as Message

SELECT partition_number,rows 
FROM sys.partitions 
WHERE object_id = OBJECT_ID('SalesPartitioned')
/*
partition_number	rows
1	2048
2	0
3	2048
4	0
5	0
6	0
*/
 
SELECT YearCol, $PARTITION.pfFiscalYear(YearCol) AS Partition, 
COUNT(*) AS [COUNT] FROM SalesPartitioned
GROUP BY $PARTITION.pfFiscalYear(YearCol),YearCol 
ORDER BY Partition;
GO

/*
YearCol	Partition	COUNT
2013	1	2048
2015	3	2048
*/


SELECT 'All done' as Message

```
