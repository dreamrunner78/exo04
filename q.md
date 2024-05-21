# Offload 
```
Offload Tpch dans une bucket en utilisant ICEBERG
```
```sql
CREATE SCHEMA IF NOT EXISTS minioiceberg.raw WITH (location = 's3a://dcp-raw-data/input')

CREATE TABLE IF NOT EXISTS minioiceberg.raw.customer1
WITH (
  format = 'PARQUET',
  location = 's3a://dcp-raw-data/input/customer/parquet'
) 
AS select * from tpch.sf1.customer

select count(*) from minioiceberg.raw.customer1


CREATE TABLE IF NOT EXISTS minioiceberg.raw.nation
WITH (
  format = 'PARQUET',
  location = 's3a://dcp-raw-data/input/nation/parquet'
) 
AS select * from tpch.sf1.nation
```

# PARQUET -> ORC
```sql
CREATE SCHEMA IF NOT EXISTS minioiceberg.derived WITH (location = 's3a://dcp-raw-data/derived')

CREATE TABLE IF NOT EXISTS minioiceberg.derived.customer1
WITH (
  format = 'ORC',
  location = 's3a://dcp-raw-data/derived/customer/orc'
) 
AS select * from minioiceberg.raw.customer1

CREATE TABLE IF NOT EXISTS minioiceberg.derived.nation
WITH (
  format = 'ORC',
  location = 's3a://dcp-raw-data/derived/nation/orc'
) 
AS select * from minioiceberg.raw.nation
```


```sql
CREATE SCHEMA IF NOT EXISTS miniocat.raw WITH (location = 's3a://dcp-raw-data/iris')
CREATE TABLE IF NOT EXISTS miniocat.raw.srcdata (
  sepal_length DOUBLE,
  sepal_width  DOUBLE,
  petal_length DOUBLE,
  petal_width  DOUBLE,
  class        VARCHAR
)   
WITH (
  external_location = 's3a://dcp-raw-data/iris',
  format = 'PARQUET'
)
select * from miniocat.raw.srcdata limit 10
```

# Iceberg snapshot (TRINO)
```sql
CREATE SCHEMA IF NOT EXISTS minioiceberg.raw WITH (location = 's3a://dcp-raw-data/raw')

CREATE TABLE IF NOT EXISTS minioiceberg.raw.customer
WITH (
  format = 'PARQUET',
  location = 's3a://dcp-raw-data/raw/customer'
) 
AS select * from tpch.sf1.customer

SELECT * FROM "minioiceberg"."raw"."customer$snapshots"

SELECT * from minioiceberg.raw.customer order by custkey asc limit 10

update minioiceberg.raw.customer set name='abe' where custkey<=10

SELECT * FROM "minioiceberg"."raw"."customer$snapshots"

SELECT * FROM "minioiceberg"."raw"."customer" 
FOR VERSION AS OF 4656274813083654134
order by custkey asc limit 20
```

# Iceberg snapshot (PRESTODB)
```sql
CREATE SCHEMA IF NOT EXISTS minioiceberg.raw WITH (location = 's3a://dcp-raw-data/raw')
drop table minioiceberg.raw.customer
CREATE TABLE IF NOT EXISTS minioiceberg.raw.customer
WITH (
  format = 'PARQUET',
  partitioning = ARRAY['nationkey'],
  location = 's3a://dcp-raw-data/raw/customer'
) 
AS select * from tpch.sf1.customer

SELECT * FROM "minioiceberg"."raw"."customer$snapshots"
SELECT * FROM "minioiceberg"."raw"."customer$properties"
SELECT * FROM "minioiceberg"."raw"."customer$history"

SELECT * from minioiceberg.raw.customer where nationkey=0

delete from minioiceberg.raw.customer where nationkey=0


SELECT * FROM "minioiceberg"."raw"."customer" 
FOR VERSION AS OF 3580219772017487607
where nationkey=0


SELECT * FROM "minioiceberg"."raw"."customer" 
FOR VERSION AS OF 1305418391854518355
where nationkey=0

```

# Time Travel Query
```sql
SELECT * FROM "iceberg"."your_schema"."your_table"
FOR TIMESTAMP AS OF TIMESTAMP '2021-01-01 12:00:00';
```


# Cache Service
```sql
CREATE SCHEMA iceberg01.bench1 WITH (location = 'alluxio://cacheqtcdcs-master-0.demo.svc.cluster.local:19998/dcpdatabucket/customer1/');
CREATE TABLE iceberg01.bench1.customer1
WITH (
  format = 'ORC',
  location = 'alluxio://cacheqtcdcs-master-0.demo.svc.cluster.local:19998/dcpdatabucket/customer1/'
) 
AS select * from tpch.sf1.customer;

CREATE SCHEMA iceberg01.bench2 WITH (location = 'alluxio://cacheqtcdcs-master-0.demo.svc.cluster.local:19998/dcpdatabucket/customer2/');

CREATE TABLE iceberg01.bench2.customer2
WITH (
  format = 'ORC',
  location = 'alluxio://cacheqtcdcs-master-0.demo.svc.cluster.local:19998/dcpdatabucket/customer2/'
) 
AS select * from tpch.sf10.customer;
```











# Query on s
```sql
CREATE SCHEMA IF NOT EXISTS cats3.schsource WITH (location = 's3a://sparklog/iris');
CREATE TABLE IF NOT EXISTS cats3.schsource.srcdata (
  sepal_length DOUBLE,
  sepal_width  DOUBLE,
  petal_length DOUBLE,
  petal_width  DOUBLE,
  class        VARCHAR
);  
WITH (
  external_location = 's3a://sparklog/iris',
  format = 'PARQUET'
);
select * from cats3.schsource.srcdata limit 10;
```
