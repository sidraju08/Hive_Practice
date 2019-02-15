# Hive_Practice

CREATE TABLE:
--------------
create database if not exists student;
show databases;
use student;
describe database student;
create table if not exists student ( name string, id int , year int);
show tables;

create table if not exists student1 ( name string, id int , year int) row format delimited fields terminated by '\t' lines terminated by '\n' stored as textfile;
create external table if not exists student3 ( name string, id int , year int) row format delimited fields terminated by '\t' lines terminated by '\n' stored as textfile;

LOAD DATA:
----------
load data local inpath '/home/hadoop/work/hive_inputs/student.txt' overwrite into table student1;
select * from student1;

load data local inpath '/home/hadoop/work/hive_inputs/student1.txt' into table student2;
select * from student2;

use student;
load data inpath '/home/hadoop/work/hive_inputs/student.txt' overwrite into table student2;


PRINT DATA:
------------
set hive.cli.print.current.db=true;
set hive.cli.print.header=true;


TABLES USING COLLECTION DATA TYPE:
-------------------------------------------

create table if not exists studentcollections 
(
name string comment 'student name', 
marks map<string,int> comment 'student marks', 
subjects array<string>, 
address struct< loc : string comment 'student location', pincode : int comment 'student pincode', city : string comment 'student city'>
) 
row format delimited 
fields terminated by '#' 
collection items terminated by ','
map keys terminated by ':'
lines terminated by '\n' 
stored as textfile;

load data local inpath '${env:HOME}/work/hive_inputs/studentcollections.txt' overwrite into table studentcollections;
select * from studentcollections;
select name, marks['math'], subjects[0], address.pincode from studentcollections;
insert overwrite local directory '/home/hadoop/work/hive_inputs/studentcollections' select * from studentcollections;


PARTITIONS:
-----------
create table if not exists partitiontable(name string, id int ) partitioned by (year int) row format delimited fields terminated by '\t' lines terminated by '\n' stored as textfile;
load data local inpath '${env:HOME}/work/hive_inputs/student1.txt' overwrite into table partiontable partition(year='1');
show partitions partitiontable;  
select * from partitiontable where year=1;


Map Join:
---------
select /*+ mapjoin(sales) */ products.* , sales.* from products join sales on products.name = sales.name;

Semi Join:
----------
select * from products left semi join sales on products.name = sales.name;


Buckets:
--------
set hive.enforce.bucketing=true;

create table bucketed_users (name string, id int) clustered by (id) into 4 buckets;

insert overwrite table bucketed_users select * from users;

select * from bucketed_users tablesample ( bucket 1 out of 2 on id);
