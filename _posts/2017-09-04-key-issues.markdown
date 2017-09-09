---
layout: post
title:  "Fixing key issues in Microsoft SQL Server after Database restore"
date:   2017-09-04 09:04:42 -0400
categories: Tech
---
<!--<h1> Background </h1>-->

The purpose of this post it to provide a solution for the issues I faced after trying to restore a database from a different environment. The purpose of this was to move modern data onto our development environment to allow for more accurate testing.

After restoring the database(DB) there were multiple issues with the master key, symmetrical key and certificates. The first error seen when trying to run a job was:

>Error = Please create a master key in the database or open the master key in the session before performing this operation.


A simple solution is to simply delete the keys and recreate the master key which would allow the symmetrical and certs to be recreated. In order to do this you must delete key dependencies going from the bottom up. The script below deletes the symmetric key, certificate, master key in that order. If you try deleting the master key first dependeny error will be thrown.

~~~~
IF EXISTS (SELECT * FROM SYS.SYMMETRIC_KEYS WHERE NAME = 'symmetric_key_name')
DROP SYMMETRIC KEY [symmetric_key_name]
GO

IF EXISTS (SELECT * FROM SYS.CERTIFICATES WHERE NAME = 'cert_name')
DROP CERTIFICATE [cert_name]
GO

DROP MASTER KEY
GO
~~~~
This script resulted in the error below. I found that the dialogue mentioned in my case was the data warehouse configuration set up to communicate with the database. 

>Cannot drop master key because dialog "xxxxxx" is encrypted by it.

The first step is to reset the service broker. This will ensure there are no connections or queries in que. 
~~~~
ALTER DATABASE db_name SET ENABLE_BROKER WITH ROLLBACK IMMEDIATE
~~~~

In my case this did not work. After much troubleshooting I found that force dropping the master key on the data warehouse solved the issue. This script will force regenerate the master key with the password specified and then drop it.
~~~~
alter master key force regenerate with encryption by password = 'enterpasswordwithrequiredspecs'  
GO        
  
drop master key  
GO        
~~~~
After this I was able to drop the master key in the database and recreate the necessary keys.
~~~~
IF NOT EXISTS (SELECT * FROM SYS.SYMMETRIC_KEYS WHERE NAME = 'name')
BEGIN

CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'password'   
END
IF NOT EXISTS (SELECT * FROM SYS.CERTIFICATES WHERE NAME = 'secret_key')
BEGIN
CREATE CERTIFICATE [secret_key]
WITH SUBJECT = 'secret_key'   
END
IF NOT EXISTS (SELECT * FROM SYS.SYMMETRIC_KEYS WHERE NAME = 'secret_key')
~~~~