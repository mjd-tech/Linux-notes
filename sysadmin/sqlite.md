# SQLite

Notes on SQLite3. Version 3.7.9

## Backup - Restore

### Backing up the database

To make a backup copy of the database, simply do a "dump" and redirect
the results to a file.

    cd /home/sqlite
    sqlite3 mydb.sqlite .dump > mydb.sql

### Restoring the database

Make sure the destination database is empty first. Alternatively you may
want to delete or rename the destination database and let sqlite create
a new one for you.

    mv mydb.sqlite mydb.sqlite.old
    sqlite3 mydb.sqlite < mydb.sql

## General Commands

**Create a SQLite Database (and Tables)**

    # sqlite3 company.db
    sqlite> create table employee(empid integer,name varchar(20),title varchar(10));
    sqlite> create table department(deptid integer,name varchar(20),location varchar(10));
    sqlite> .quit

company.db gets created if it does not exist.

**Insert Records**

You can execute all the insert statements from the sqlite command line,
or you can add those commands into a file and execute the file as shown
below.

First, create a insert-data.sql file as shown below.

    # vi insert-data.sql
    insert into employee values(101,'John Smith','CEO');
    insert into employee values(102,'Raj Reddy','Sysadmin');
    insert into employee values(103,'Jason Bourne','Developer');
    insert into employee values(104,'Jane Smith','Sale Manager');
    insert into employee values(105,'Rita Patel','DBA');

    insert into department values(1,'Sales','Los Angeles');
    insert into department values(2,'Technology','San Jose');
    insert into department values(3,'Marketing','Los Angeles');

The following will execute all the commands from the insert-data.sql in
the company.db database

    # sqlite3 company.db < insert-data.sql

**View Records**

Once you’ve inserted the records, view it using select command as shown
below.

    # sqlite3 company.db
    sqlite> select * from employee;
    101|John Smith|CEO
    102|Raj Reddy|Sysadmin
    103|Jason Bourne|Developer
    104|Jane Smith|Sale Manager
    105|Rita Patel|DBA

    sqlite> select * from department;
    1|Sales|Los Angeles
    2|Technology|San Jose
    3|Marketing|Los Angeles

**Add a Column to an Existing Table**

The following examples adds deptid column to the existing employee
table;

    sqlite> alter table employee add column deptid integer;

Update the department id for the employees using update command as shown
below.

    update employee set deptid=3 where empid=101;
    update employee set deptid=2 where empid=102;
    update employee set deptid=2 where empid=103;
    update employee set deptid=1 where empid=104;
    update employee set deptid=2 where empid=105;

**View all Tables in a Database** Execute the following command to view
all the tables in the current database.

    sqlite> .tables
    dept      employee

**More complex select command**

    sqlite> select * from employee where empid >= 102 and empid  select * from dept where location like 'Los%';
    1|Sales|Los Angeles
    3|Marketing|Los Angeles

**Search and Replace**

    update table_name set 
      field_name = replace(field_name, 'old_value', 'new_value');

Can also use option where clause:

    update table_name set 
      field_name = replace(field_name, 'old_value', 'new_value')
    where
      field_name like '%some_text%';

