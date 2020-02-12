# Advanced SQL Workshop Notes

- [Notes](https://github.com/laurosilvacom/advanced-sql-workshop/blob/master/notes.md)

## System Requirements & Setup

- [x] Clone this repo:

```shell
git clone https://github.com/twclark0/sql-advanced.git

cd sql-advanced
```

- [x] Install postgres on your machine. When it's install, make sure you can connect to your local DB by running `psql postgres` in your terminal

- [x] Use HomeBrew `brew install postgresql`

- [x] Once you have postgres running locally, run the following commands:

```sql
create table users (
user_handle uuid Primary key,
first_name text,
last_name text,
email text
);

```

```sql
create table purchases (
date date,
user_handle uuid,
sku uuid,
quantity int
);

```

```sql
create table members (
start_date date,
end_date date,
user_handle uuid,
first_name text,
email text
);

```
