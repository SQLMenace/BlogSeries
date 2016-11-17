You can run this whole block of code in 1 shot <br>
Blog post: What's new in SQL Server 2016: CREATE OR ALTER 

```SQL

USE tempdb
GO

--old way of dropping a proc then having a create script
IF OBJECT_ID('procTest') is not null
DROP PROCEDURE procTest
GO

CREATE PROCEDURE procTest
AS
BEGIN
 PRINT (1)
END;
GO
--another way is to have a dummy proc created, 
--that way your alter part is always the same
IF OBJECT_ID('dbo.procTest') IS NULL
  EXEC ('CREATE PROCEDURE dbo.procTest AS RETURN 0;')
GO

ALTER PROCEDURE  procTest
AS
BEGIN
 PRINT (1)
END;
GO

-- the easier way in sql server 2016 SP1 and up
CREATE OR ALTER PROCEDURE procTest
AS
BEGIN
 PRINT (1)
END;
GO

-- works the same with functions
CREATE OR ALTER FUNCTION fnTest()
RETURNS INT
AS
BEGIN
 RETURN(1)
END;
GO

-- also works with views
CREATE OR ALTER VIEW vwTest
AS
 SELECT 1 AS col;
GO

-- first we need a table for the trigger
IF NOT EXISTS(SELECT * FROM sys.tables WHERE name = 'BooFar')
CREATE TABLE BooFar(id int)
GO

-- works with triggers
CREATE OR ALTER TRIGGER trTest 
ON BooFar 
AFTER INSERT, UPDATE 
AS
 RAISERROR ('Hello from trigger', 1, 10);
 GO

 -- just a test to make sure the trigger works
 INSERT BooFar values(1)

 -- you should see this in the message tab
 /*
Hello from trigger
Msg 50000, Level 1, State 10

(1 row(s) affected)
*/


-- Also works with DDL triggers

CREATE OR ALTER TRIGGER safety   
ON DATABASE   
FOR DROP_TABLE, ALTER_TABLE   
AS   
   PRINT 'You must disable Trigger "safety" to drop or alter tables!'   
   ROLLBACK;  
   
   
--  drop everything by using
--  DROP object IF EXISTS
 DROP TABLE IF EXISTS  BooFar
 DROP PROCEDURE IF EXISTS  procTest
 DROP VIEW IF EXISTS  vwTest
 DROP TRIGGER IF EXISTS  safety

```
