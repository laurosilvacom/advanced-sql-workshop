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
