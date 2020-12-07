## Find minimum database permissions used by an existing system.

### Problem
We connect to a database using a user with its name and password. When installing a program this user might need highest permission on the database so that it can create tables, views, ect. However after installing it may not need all that permissions anymore, so it should be limited to the minimumn permissions needed. Our problem is to find the mininum permissions needed by a user on an existing system. We will try to solve this problem on both SQL Server and Oracle

### Method
In fact we may research the system first to determine the minimum permissons theoretically and then test that by the following method.

The user is first given the highest perminission. The permissions might be specified by the program. If not we can use the highest permission for a user on a database system.
Install the program with that permissions and then reduce user permissions to the one we determined already. We then verify that the system is working properly. If it is not, we might need to give the user more permission.

Below are scripts that can be used to perform this method

***SQL Server***
  1. Creates a login named test1 (if it does not exsit)
  2. Creates our test database
  3. Create a user named user1 which has a permission of db_owner and associates with login test1

```sql
-- Create a login named test1
USE master;
IF NOT EXISTS(SELECT * FROM sys.syslogins WHERE name = 'test1')
BEGIN
    CREATE LOGIN [test1] WITH PASSWORD=N'test1', DEFAULT_DATABASE=[master], DEFAULT_LANGUAGE=[us_english], CHECK_EXPIRATION=ON, CHECK_POLICY=ON
END
GO

-- Create the database
USE master;
CREATE DATABASE <our database>;
GO

-- In this database, create a user named user1 associated with login test1 and grant db_owner permission to it
USE kafc_data;
CREATE USER user1 FOR LOGIN test1 WITH DEFAULT_SCHEMA=dbo;
ALTER ROLE db_owner ADD MEMBER user1;
GO
```

After installing the system with the above login, reduce permissions on the associated user in the database.
In my case, the system needed 2 predefined permissions db_datareader, db_datawriter and 2 other permissions CREATE TABLE and ALTER
```sql
USE <our database>;
ALTER ROLE db_owner DROP MEMBER user1;     -- Remove db_owner permission from user1
ALTER ROLE db_datawriter ADD MEMBER user1; -- The user can write(insert, update, delete) data to/from database
ALTER ROLE db_datareader ADD MEMBER user1; -- The user can read data from database

CREATE ROLE role1;
GRANT CREATE TABLE TO role1; -- Permission to create a table
GRANT ALTER TO role1;        -- Permission to create an index

ALTER ROLE role1 ADD MEMBER user1;
GO
```

Finally tests the system to make sure that it still works with new permissions

***Oracle***

Connect to database with sysdba
```
connect / as sysdba
```

Create the database and a user with all privilages
```sql
-- DROP TABLESPACE our_tablespace INCLUDING CONTENTS;
CREATE TABLESPACE our_tablespace 
   DATAFILE 'our_database.dbf' 
   SIZE 500m
   AUTOEXTEND ON;

CREATE USER test1
IDENTIFIED BY password1
DEFAULT TABLESPACE our_tablespace 
QUOTA UNLIMITED ON our_tablespace;
GRANT ALL PRIVILEGES TO test1;
```

Install the system with user test1

Reduce permissions of the the user
```sql
REVOKE ALL PRIVILEGES FROM test1;
GRANT CREATE SESSION, CREATE TABLE, CREATE PROCEDURE, CREATE SEQUENCE TO test1;
```

Test the system to make sure it is working properly

### References
- https://docs.microsoft.com/en-us/sql/relational-databases/security/authentication-access/server-level-roles?view=sql-server-ver15
- https://docs.microsoft.com/en-us/sql/relational-databases/security/authentication-access/database-level-roles?view=sql-server-ver15
- https://docs.oracle.com/cd/B19306_01/network.102/b14266/admusers.htm#DBSEG10000
- https://docs.oracle.com/database/121/TTSQL/privileges.htm#TTSQL338

