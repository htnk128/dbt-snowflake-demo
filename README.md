# 🥪 The Jaffle Shop 🦘

_powered by the dbt Fusion engine_

Welcome! This is a sandbox project for exploring the basic functionality of Fusion. It's based on a fictional restaurant called the Jaffle Shop that serves [jaffles](https://en.wikipedia.org/wiki/Pie_iron).

To get started:
1. Set up your database connection in `~/.dbt/profiles.yml`. If you got here by running `dbt init`, you should already be good to go.
2. Run `dbt build`. That's it!

> [!NOTE]
> If you're brand-new to dbt, we recommend starting with the [dbt Learn](https://learn.getdbt.com/) platform. It's a free, interactive way to learn dbt, and it's a great way to get started if you're new to the tool.

## Create a Demo Environment
```sql
USE ROLE accountadmin;

create warehouse if not exists DEMO_DBT_WH with warehouse_size = 'XSMALL';
create database if not exists DEMO_DBT;
create schema if not exists DEMO_DBT.DEV;
create schema if not exists DEMO_DBT.DEV_RAW;
```

## Delete a Demo Environment
```sql
USE ROLE accountadmin;

drop warehouse if exists DEMO_DBT_WH;

drop database if exists DEMO_DBT;
```
