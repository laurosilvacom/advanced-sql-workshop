These notes follow the same structure as the workshop (1-10). Each lesson includes short-snippets of constructive feedback.

# Table of Contents

1. [Bulk insert / update / export](#1-bulk-insert--update--export)
2. [The insert on conflict flag](#2-the-insert-on-conflict-flag)
3. [Casting types](#3-casting-types)
4. [Defining custom types](#4-defining-custom-types)
5. [Query performance tuning](#5-query-performance-tuning)
6. [Creating Common Table Expressions](#6-creating-common-table-expressions-cte)
7. [Filter aggregated data with `having`](#7-filter-aggregated-data-with-having)
8. [Defining scopes and variables with `do` and `declare`](#8-defining-scopes-and-variables-with-do-and-declare)
9. [Conditional returns with `case`, `when`, `then`, `else`, & `end`](#9-conditional-returns-with-case-when-then-else--end)
10. [Perform Multiple Steps in One with Transactions](#10-perform-multiple-steps-in-one-with-transactions)

# Notes

## 1. Bulk insert / update / export

```sql
create temporary table temp_users (user_handle uuid, first_name text, last_name text, email text);

copy temp_users(user_handle,first_name,last_name,email) FROM '/Users/laurosilvacom/Documents/clients/eggheadio/advanced-sql-workshop/sql-advanced-users.csv' DELIMITER ',' CSV HEADER;

update users set email = temp_users.email from temp_users where users.user_handle = temp_users.user_handle;

drop table temp_users;
```

<details>

<summary><b>Feedback</b></summary>

- Off to a really good start, easy to follow the exercice.
  </details>

## 2. The insert on conflict flag

```sql
insert into users (user_handle, first_name, last_name, email) values ('57bd72d4-7115-11e9-a923-1681be663d3e', 'Lucie', 'Jones', 'Lucie-Jones@gmail.com');
```

```sql
select * from users where first_name = 'Lucie';
```

then do a insert if not found or an update with the new information.

```sql
insert into users (user_handle, first_name, last_name, email) values ('57bd72d4-7115-11e9-a923-1681be663d3e', 'Lucie', 'Jones', 'Lucie-Jones@gmail.com') on conflict do nothing;

```

```sql
insert into users (user_handle, first_name, last_name, email) values ('57bd72d4-7115-11e9-a923-1681be663d3e', 'Lucie', 'Jones', 'Lucie-Jones@gmail.com') on conflict (user_handle) do update set first_name = excluded.first_name, last_name = excluded.last_name, email = excluded.email;

```

<details>

<summary><b>Feedback</b></summary>

- It would be good to add here another use case scenario for this lesson.

</details>

## 3. Casting types

- Two types can be binary coercible, which means that the conversion can be performed "for free" without invoking any function.
- This requires that corresponding values use the same internal representation.

`CAST (source_type AS target_type) WITH FUNCTION function_name (argument_type [, ...]) [ AS ASSIGNMENT | AS IMPLICIT ]`

```sql
cast (expression as target_type)

```

```sql
select cast (now() as date);

select cast ('100' as integer);

```

- PostgreSQL only type cast :: operator

```sql
select now()::date;
```

```sql

create temporary table users_temp (create_date date, user_handle uuid, first_name text, last_name text, email text);

insert into users_temp (create_date, user_handle, last_name, first_name) values (now(), uuid_generate_v4(), 'jones', 'michelle');

select create_date from users_temp u inner join (select now() as date) n on u.create_date = n.date;

```

<details>

<summary><b>Feedback</b></summary>

- Add this to the exercise: Try adding extension CREATE EXTENSION IF NOT EXISTS "uuid-ossp"; As per https://stackoverflow.com/questions/12505158/generating-a-uuid-in-postgres-for-insert-statement

</details>

## 4. Defining custom types

- `CREATE TYPE` creates a new data type to use in database
- Composite, Enumerated, Range, Base, & Array types

`CREATE TYPE name AS ENUM ( [ 'label' [, ... ] ] )`

```sql
create type user_status as enum ('member', 'instructor', 'developer');
```

```sql
alter table users add column status user_status;
```

```sql
update users set status = 'instructor' where first_name = 'Lucie';
```

<details>

<summary><b>Feedback</b></summary>

- Really enjoyed this lesson. Spent enough time answering questions.

</details>

## 5. Query performance tuning

- Seq scan

```sql
explain select * from users;
```

- Index scan

```sql
explain select * from users where user_handle = 'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11';
```

- Aggregate and seq scan

```sql
explain select sum(quantity) from purchases where date < now();
```

- Explain & Analyze

```sql
explain analyze select * from users where user_handle = 'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11';
```

- Hash node shows hash buckets, batches, and peak amount of memory used for the hash table

```sql
explain analyze select * from users inner join purchases using(user_handle);
```

vs.

```sql
explain analyze select * from users inner join (select * from purchases) s using(user_handle);
```

```
copy purchases FROM '/Users/laurosilvacom/Documents/clients/eggheadio/advanced-sql-workshop/sql-advanced-purchases.csv' DELIMITER ',' CSV HEADER;
```

<details>

<summary><b>Feedback</b></summary>

- In my personal experience I had a hard time understanding the difference between Aggregate and Cost at the start of the lesson.

</details>

## 6. Creating Common Table Expressions (CTE)

- CTE: Common Table Expression
- Created using `with` (thought of as defining temporary tables)
- Can use any CRUD statement within the `with` which can be within any CRUD query

`with cte_name as ( [select, delete query] )`

```sql

with dates as (select now() as date)

select * from dates;
```

#### A member history table. Contains a record of every event of a member

1.  A Member's _current_ email (Email can change over the years)
2.  Their start_date of when they originally bought a license

```sql

insert into members values ('2018-01-01',  '2019-01-01', '57bd8cd8-7115-11e9-a923-1681be663d3e', 'tyler', 'orignal@gmail.com');
insert into members values ('2019-02-20', null, '57bd8cd8-7115-11e9-a923-1681be663d3e', 'joe', 'new@gmail.com');

with most_recent_membership as (
    select distinct on (user_handle) user_handle, email
        from members
    order by user_handle, start_date desc
)

select mr.email, min(m.start_date) from members m
left outer join most_recent_membership mr using(user_handle) group by mr.email;

```

##### Deleting

```sql
create table purhases_copy (
    date date,
    user_handle uuid,
    sku uuid,
    quantity int
);

with moved_purchases as (
    delete from purchases
    RETURNING *
)
insert into purhases_copy select * from moved_purchases;
```

<details>

<summary><b>Feedback</b></summary>

- In general, don't use a text editor to show the content. It makes it hard to follow with all the markdown syntax.
- I really enjoyed how he added real world cases when using a CTE.

</details>

## 7. Filter aggregated data with `having`

`having` then aggregate

#### Exercise

```sql
insert into purchases values ('2019-02-02', 'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11', uuid_generate_v4(), 1);
insert into purchases values ('2019-02-03', 'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11', uuid_generate_v4(), 2);
insert into purchases values ('2019-02-04', 'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11', uuid_generate_v4(), 3);
insert into purchases values ('2019-04-20', 'cebfb6c1-7e85-44b1-8408-4084d38f87dd', uuid_generate_v4(), 3);

select user_handle, sum(quantity) as total from purchases where sum(quantity) > 5 group by user_handle;

select user_handle, sum(quantity) as total from purchases group by user_handle having sum(quantity) > 5;

```

<details>

<summary><b>Feedback</b></summary>

- Really enjoy this lesson, easy to follow the concepts and the exercise.

</details>

## 8. Defining scopes and variables with `do` and `declare`

```sql
do $$
declare
    handle uuid := '099ab71b-53ad-4fc6-823a-5f22aabc2d45';
    createDate date;
begin
    select create_date into createDate from users where user_handle = handle;
    insert into members values (createDate, null, handle, 'Stacy', 'freemon');
end $$;
```

###### If blocks

```sql
do $$
declare
    handle uuid := '17c1ed46-fb34-4c81-ae16-85324da7e1ec';
    createDate date;
begin
    select create_date into createDate from users where user_handle = handle;
    if (createDate is not null) then
        insert into members values (createDate, null, handle, 'Kaitlin', 'Johnson');
    end if;
end $$;
```

###### Functions around variables referencing another variable

```sql
$$
declare
    handle uuid := '3ebf30fd-f524-4ec6-b864-9cb2ae57f5ed';
    someDate date := '2019-04-01';
    createDate date := least('2019-01-12', someDate);
begin
    insert into members values (createDate, null, handle, 'Jason', 'Anderson');
end;
$$
```

<details>

<summary><b>Feedback</b></summary>

- Really enjoy this lesson, easy to follow.

</details>

## 9. Conditional returns with `case`, `when`, `then`, `else`, & `end`

- Can be used wherever an expression is valid
- If no else then returns `NULL`
- Close `case` with `end`

`CASE WHEN condition THEN result [WHEN ...] [ELSE result] END`

```sql
select first_name,
    case when status is null then 'member' else status end
from users;
```

```sql
select first_name, start_date
from members
where case when email is not null then start_date > '2019-01-01' end;
```

<details>

<summary><b>Feedback</b></summary>

- All good on this lesson. Spent enough time answering questions.

</details>

## 10. Perform Multiple Steps in One with Transactions

- Keywords are `begin`, `commit`, `rollback`, `savepoint`, `start transaction`
- If any fail then all rollback by default

- `begin`
- `commit`
- `rollback`
- `savepoint`
- `start transaction` (optional)

```sql
start transaction;
insert into purchases values ('2019-05-20', 'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11', uuid_generate_v4(), 1);
update purchases set quantity = 5 where date = '2019-05-20';
commit;
```

```sql
begin;
    insert into purchases values ('2019-10-10', 'f0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11', uuid_generate_v4(), 1);
    savepoint insert_save_point;
    insert into purchases values ('2019-05', 'fq0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11', uuid_generate_v4(), 1);
    rollback to insert_save_point;
    update purchases set quantity = 8;
commit;
```

<details>

<summary><b>Feedback</b></summary>

- Overall a really good course! 🙌

</details>
