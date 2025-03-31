---
author: alc
category:
  - analytics
  - tips
cover:
  alt: dmssql
  image: /wp-content/uploads/2014/03/dmssql.png
date: "2014-03-13T08:48:47+00:00"
guid: http://laubplusco.net/?p=832
title: Sitecore DMS visits from specific IP addresses
url: /sitecore-dms-visits-specific-ip-addresses/

---
Sitecore DMS stores IP addresses as a varbinary(16) in the Visits table in the analytics database.

This is just perfect but it is impossible to read the IP addresses as a normal human being. To help out with this I found the following SQL function which can convert an IPv4 address to a binary.

```tsql
CREATE FUNCTION dbo.fnBinaryIPv4(@ip AS VARCHAR(15)) RETURNS BINARY(4)
AS
BEGIN
	DECLARE @bin AS BINARY(4)

	SELECT @bin = CAST( CAST( PARSENAME( @ip, 4 ) AS INTEGER) AS BINARY(1))
				+ CAST( CAST( PARSENAME( @ip, 3 ) AS INTEGER) AS BINARY(1))
				+ CAST( CAST( PARSENAME( @ip, 2 ) AS INTEGER) AS BINARY(1))
				+ CAST( CAST( PARSENAME( @ip, 1 ) AS INTEGER) AS BINARY(1))

	RETURN @bin
END
```

So let say that you want to delete all visits from localhost, simply run the following SQL statement using Sql Server Management Studio.

```tsql
USE [ANALYTICS DATABASENAME];

IF NOT EXISTS (SELECT 1 FROM sys.objects WHERE type = 'FN' AND name = 'fnBinaryIPv4')
BEGIN
    DECLARE @sql NVARCHAR(MAX);
    SET @sql = N'CREATE FUNCTION dbo.fnBinaryIPv4(@ip AS VARCHAR(15)) RETURNS BINARY(4)
	AS
	BEGIN
		DECLARE @bin AS BINARY(4)

		SELECT @bin = CAST( CAST( PARSENAME( @ip, 4 ) AS INTEGER) AS BINARY(1))
					+ CAST( CAST( PARSENAME( @ip, 3 ) AS INTEGER) AS BINARY(1))
					+ CAST( CAST( PARSENAME( @ip, 2 ) AS INTEGER) AS BINARY(1))
					+ CAST( CAST( PARSENAME( @ip, 1 ) AS INTEGER) AS BINARY(1))

		RETURN @bin
	END';
    EXEC sp_executesql @sql;
END
go

Delete from [Visits] where [IP] = dbo.fnBinaryIPv4('127.0.0.1');
```

Converting the binary back to human readable format is done like this:

```tsql
CREATE FUNCTION dbo.fnDisplayIPv4(@ip AS BINARY(4)) RETURNS VARCHAR(15)
AS
BEGIN
    DECLARE @str AS VARCHAR(15)

    SELECT @str = CAST( CAST( SUBSTRING( @ip, 1, 1) AS INTEGER) AS VARCHAR(3) ) + '.'
                + CAST( CAST( SUBSTRING( @ip, 2, 1) AS INTEGER) AS VARCHAR(3) ) + '.'
                + CAST( CAST( SUBSTRING( @ip, 3, 1) AS INTEGER) AS VARCHAR(3) ) + '.'
                + CAST( CAST( SUBSTRING( @ip, 4, 1) AS INTEGER) AS VARCHAR(3) );

    RETURN @str
END;
go
```

So to get a human readable list of all IP's in the Analytics database write something like this:

```tsql
USE [ANALYTICS DATABASENAME];

IF NOT EXISTS (SELECT 1 FROM sys.objects WHERE type = 'FN' AND name = 'fnDisplayIPv4')
BEGIN
    DECLARE @sql NVARCHAR(MAX);
    SET @sql = N'CREATE FUNCTION dbo.fnDisplayIPv4(@ip AS BINARY(4)) RETURNS VARCHAR(15)
				AS
				BEGIN
					DECLARE @str AS VARCHAR(15)

					SELECT @str = CAST( CAST( SUBSTRING( @ip, 1, 1) AS INTEGER) AS VARCHAR(3) ) + ''.''
								+ CAST( CAST( SUBSTRING( @ip, 2, 1) AS INTEGER) AS VARCHAR(3) ) + ''.''
								+ CAST( CAST( SUBSTRING( @ip, 3, 1) AS INTEGER) AS VARCHAR(3) ) + ''.''
								+ CAST( CAST( SUBSTRING( @ip, 4, 1) AS INTEGER) AS VARCHAR(3) );

					RETURN @str
				END';
    EXEC sp_executesql @sql;
END

select dbo.fnDisplayIPv4([IP]) from Visits
```

That was it, hope it helps someone out there.
