## problem 1
<pre><code>show databases;
create database problem1;

create external table if not exists problem1.account
(id string, status string, amount int, type string)
row format delimited
fields terminated by ','
stored as textfile
location '/user/training/problem1';

select a.id, a.type, a.status, a.amount, (a.amount-b.average) as difference
from problem1.account a
join ( select type, avg(amount) as average 
      from problem1.account where status = "Active" group by type) b
on a.type = b.type
where a.status like 'Active';</pre></code>



## problem2
<pre><code>create database problem2;

create table if not exists problem2.employee
(id int, fname string, lname string, address string, city string, state string, zip string, birthday string, hireday string)
stored as parquet
location '/user/training/problem2/data/employee/';

select * from problem2.employee limit 3;</pre></code>

## problem3 
<pre><code>create database problem3;

create table if not exists problem3.account
(id int, fname string, lname string, hphone string)
row format delimited
fields terminated by ','
stored as textfile
location '/user/training/problem3/customer';

create table if not exists problem3.customer
(id int, amount int)
row format delimited
fields terminated by ','
stored as textfile
location '/user/training/problem3/account';

select * from problem3.account limit 3;
select * from problem3.customer limit 3;

create table problem3.metastore as
select a.id, a.fname, a.lname, a.hphone 
from problem3.account a
inner join (select id, amount from problem3.customer) b
on a.id = b.id
where b.amount < 0;

select * from problem3.metastore limit 3;</pre></code>

## problem4 
<pre><code>create database problem4;

create external table if not exists problem4.origin
(id int, lname string, fname string, address string, city string, state string, zipcode string)
row format delimited
fields terminated by '\t'
stored as textfile
location '/user/training/problem4/data/employee1/';

create external table if not exists problem4.new
(id int, dnum int, fname string, lname string, address string, city string, state string, zipcode string)
row format delimited
fields terminated by ','
stored as textfile
location '/user/training/problem4/data/employee2/';

select * from problem4.origin limit 3;
select * from problem4.new limit 3;

insert overwrite directory '/user/training/problem4/solution/'
row format delimited
fields terminated by '\t'
select * from (
select id, fname, lname, address, city, state, split(zipcode,'-')[0] as zipcode
from problem4.origin where state = 'CA'
union all
select id, concat(substr(fname,0,1),substr(lower(fname),2)) as fname, concat(substr(lname,0,1),substr(lower(lname),2)) as lname, address, city, state, zipcode from problem4.new where state = 'CA'
) unioned;</pre></code>

## problem5 
<pre><code>create database problem5;

create external table if not exists problem5.employee
(id int,fname string,lname string,address string,city string,state string,zipcode string)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION '/user/training/problem5/employee';

create external table if not exists problem5.customer
(id int,fname string,lname string,address string,city string,state string,zipcode string)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION '/user/training/problem5/customer';

insert overwrite directory '/user/training/problem5/solution/'
row format delimited
fields terminated by '\t'
select * from (
select * from problem5.customer where city = 'Palo Alto' and state = 'CA'
union all
select * from problem5.employee where city = 'Palo Alto' and state = 'CA'
) unioned;</pre></code>

## problem6
<pre><code>create database problem6;

create external table if not exists problem6.employee
(id int,fname string,lname string,address string,city string,state string,zipcode string, birthday string)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION '/user/training/problem6/employee';

select * from problem6.employee limit 3;

create table problem6.solution as
select id, fname, lname, address, city, state, zipcode, substr(birthday,0,5) as birthday 
from problem6.employee;

select * from problem6.solution limit 3;</pre></code>

## problem7
<pre><code>create database problem7;

create external table if not exists problem7.employee
(id int,fname string,lname string,address string,city string,state
string,zipcode string, birthday string)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION '/user/training/problem7/employee';

select * from problem7.employee;

select concat(fname,' ',lname) as name from problem7.employee
where city='Seattle'
order by name;</pre></code>

## problem8
<pre><code>create database problem8;

sqoop export --connect jdbc:mysql://localhost/problem8 \
  --username cloudera \
  --password cloudera \
  --table solution \
  --input-fields-terminated-by '\t' \
  --export-dir hdfs://localhost:8020/user/training/problem8/data/customer

mysql -u cloudera -p
mysql> select * from problem8.solution limit 3;</pre></code>

## problem9
<pre><code>create database problem9;

create external table if not exists problem9.customer
(id int,fname string,lname string,address string,city string,state
string,zipcode string, birthday string)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION '/user/training/problem9/customer';

create table problem9.solution as
select concat('A',cast(id as string)),fname,lname,address,city,state,zipcode
from problem9.customer;</pre></code>

## problem10
<pre><code>create database problem10;

create table problem10.customer as
select * from problem9.customer;

create external table if not exists problem10.billing
(id int,charge decimal(5,2),billdata timestamp)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION '/user/training/problem10/billing';

select * from problem10.customer limit 3;
select * from problem10.billing limit 3;

create table problem10.solution as 
select a.id, a.fname, a.lname, a.city, a.state, b.charge, split(b.billdata,' ')[0] as billdata
from problem10.customer a
join problem10.billing b
on a.id = b.id;

select * from problem10.solution limit 3;</pre></code>

## problem11 
<pre><code>create database problem11;
/* probpem11.a */ 
select * from default.products limit 10;
select * from default.order_details limit 10;
select * from default.orders limit 10;

SELECT c.name, count(*) 
FROM default.orders a, default.order_details b, default.products c 
WHERE a.order_id = b.order_id AND b.prod_id = c.prod_id AND c.brand = 'Dualcore' 
GROUP BY c.name LIMIT 3;

/* probpem11.b */
SELECT to_date(a.order_date) as date, sum(c.price) as revenue, sum(c.price-c.cost) as profit  
FROM default.orders a, default.order_details b, default.products c 
WHERE a.order_id = b.order_id AND b.prod_id = c.prod_id AND c.brand = 'Dualcore' 
group by to_date(a.order_date);

/* probpem11.c */
SELECT a.order_id, sum(c.price) as total_amount  
FROM default.orders a, default.order_details b, default.products c 
WHERE a.order_id = b.order_id AND b.prod_id = c.prod_id
group by a.order_id 
order by total_amount 
limit 10;</pre></code>

