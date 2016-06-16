/*
> Answer to http://stackoverflow.com/q/10914576/4228193 (t-sql-split-string)<br />
> Based on answers from Andy Robinson (http://stackoverflow.com/a/10914602/4228193) and AviG (http://stackoverflow.com/a/13296353/4228193)<br />
> Enhanced functionality ref: http://stackoverflow.com/q/2025585/4228193 (LEN function not including trailing spaces in SQL Server)<br />
> This file should be valid as both a markdown file and an SQL file<br />

```

*/

CREATE FUNCTION dbo.splitstring ( --CREATE OR ALTER
    @stringToSplit NVARCHAR(MAX)
) RETURNS @returnList TABLE ([Item] NVARCHAR (MAX))
AS BEGIN
    DECLARE @name NVARCHAR(MAX)
    DECLARE @pos BIGINT
    SET @stringToSplit = @stringToSplit + ','             -- this should allow entries that end with a `,` to have a blank value in that "column"
    WHILE ((LEN(@stringToSplit+'_') > 1)) BEGIN           -- `+'_'` gets around LEN trimming terminal spaces. See URL referenced above
        SET @pos = COALESCE(NULLIF(CHARINDEX(',', @stringToSplit),0),LEN(@stringToSplit+'_')) -- COALESCE grabs first non-null value
        SET @name = SUBSTRING(@stringToSplit, 1, @pos-1)  --MAX size of string of type nvarchar is 4000 
        SET @stringToSplit = SUBSTRING(@stringToSplit, @pos+1, 4000) -- With SUBSTRING fn (MS web): "If start is greater than the number of characters in the value expression, a zero-length expression is returned."
        INSERT INTO @returnList SELECT @name --additional debugging parameters below can be added
        -- + ' pos:' + CAST(@pos as nvarchar) + ' remain:''' + @stringToSplit + '''(' + CAST(LEN(@stringToSplit+'_')-1 as nvarchar) + ')'
    END
    RETURN
END
GO

/*
```
Test cases

`SELECT *,LEN(Item+'_')-1 'L' from splitstring('a,,b')` --see URL referenced as "enhanced functionality" above

Item  | L
--- | ---
a |  1
  |  0
b |  1

`SELECT *,LEN(Item+'_')-1 'L' from splitstring('a,,')`

Item  | L   
--- | ---
a |  1
  |  0
  |  0

`SELECT *,LEN(Item+'_')-1 'L' from splitstring('a,, ')`

Item  | L   
--- | ---
a |  1
  |  0
  |  1

`SELECT *,LEN(Item+'_')-1 'L' from splitstring('a,, c ')`

Item  | L   
--- | ---
a |  1
  |  0
 c | 3
 
 */
