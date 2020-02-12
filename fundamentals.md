# SQL Fundamentals Notes

## Run postgres:

```bash
$ psql postgres
psql (11.2)
Type "help" for help.

$ postgres=#
```

## Add Data to a Table with SQL Insert

```sql
insert into Users (create_date, user_handle, last_name, first_name ) values ('2018-06-06', 'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11', 'clark', 'tyler');
```

With built in postgres functions:

```sql
insert into Users (create_date, user_handle, last_name, first_name ) values (now(), 'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11', 'johnson', 'patrick');
```

Shorthand:

```sql
insert into Users values (now(), 'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11', 'jones', 'zac');
```

## Pull a table:

```sql
postgres=# select * from NAME_OF_TABLE;

postgres=# select * from Users;
postgres=# select * from purchases;
postgres=# select * from members;
```

## Count number of tables

```sql
select count(*) from members;
```

## Updating Data

```sql
update Users set last_name = 'berry', first_name = 'jerry' where last_name = 'clark';
```

| create_date              | user_handle                          | last_name | first_name |
| ------------------------ | ------------------------------------ | --------- | ---------- |
| 2018-06-06T00:00:00.000Z | a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11 | jones     | debbie     |
| 2018-06-06T00:00:00.000Z | a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11 | berry     | jerry      |
| 2018-06-06T00:00:00.000Z | a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11 | berry     | jerry      |

---

### Delete a Row

```sql
delete from Users where last_name = 'Jones';
```

### Delete all data in a table

```sql
truncate table Users;
```

### Delete the Table

```sql
drop table Users;
```

### Create an index on the Users Table

```
create index test1_user_handle_index ON Users (user_handle);
```

### Delete an index on the Users Table

```
drop index test1_user_handle_index;
```

### Create a new table

```sql
create table Purchases (date date, user_handle uuid, sku uuid, quantity int);
```

### Create a row of data with user_handle

```sql
insert into Purchases (date, user_handle, sku, quantity) values ('2018-12-12', 'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11', uuid_generate_v4(), 2);
```

### Combine the User and Purchases table user handles

```sql
select * from Users u left outer join Purchases p on u.user_handle = p.user_handle;
```

### Create a row of data with random id's

```sql
insert into Purchases values ('2019-02-02', uuid_generate_v4(), uuid_generate_v4(), 1);
```

### Cross join the Users and Purchases table

```sql
select * from Users u cross join Purchases p;
```

### Combine the User and Purchases table emails

```sql
select * from Users u left outer join Purchases ua on u.email = ua.email;
```
