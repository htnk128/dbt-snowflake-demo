# dbt-snowflake-demo

## サンドボックス環境作成
```
USE ROLE accountadmin;

-- ウェアハウス、データベース、スキーマの作成
create warehouse if not exists SHINEE_SANDBOX_WH with warehouse_size = 'XSMALL';
create database if not exists SHINEE_SANDBOX_RAW;
create database if not exists SHINEE_SANDBOX_MART;
create schema if not exists SHINEE_SANDBOX_RAW.jaffle_shop;

USE WAREHOUSE SHINEE_SANDBOX_WH;

--サンプルデータ用テーブル作成とサンプルデータ取込
create table SHINEE_SANDBOX_RAW.jaffle_shop.customers
( id integer,
  first_name varchar,
  last_name varchar
);

create table SHINEE_SANDBOX_RAW.jaffle_shop.orders
( id integer,
  user_id integer,
  order_date date,
  status varchar,
  _etl_loaded_at timestamp default current_timestamp
);

copy into SHINEE_SANDBOX_RAW.jaffle_shop.customers (id, first_name, last_name)
from 's3://dbt-tutorial-public/jaffle_shop_customers.csv'
file_format = (
    type = 'CSV'
    field_delimiter = ','
    skip_header = 1
    ); 


copy into SHINEE_SANDBOX_RAW.jaffle_shop.orders (id, user_id, order_date, status)
from 's3://dbt-tutorial-public/jaffle_shop_orders.csv'
file_format = (
    type = 'CSV'
    field_delimiter = ','
    skip_header = 1
    );


--データが正しく取り込まれているか確認
select * from SHINEE_SANDBOX_RAW.jaffle_shop.customers;
select * from SHINEE_SANDBOX_RAW.jaffle_shop.orders;
```

## サンドボックス環境削除
```
USE ROLE accountadmin;

-- サンドボックス環境の削除
-- データベースを削除すると、その中のスキーマ・テーブル・データもすべて削除されます

-- ウェアハウスの削除
drop warehouse if exists SHINEE_SANDBOX_WH;

-- データベースの削除（スキーマ・テーブルも含めて削除）
drop database if exists SHINEE_SANDBOX_RAW;
drop database if exists SHINEE_SANDBOX_MART;
```
