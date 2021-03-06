One of my personal favorite aspects of SQL is the ability to write queries within a query. This composability makes it easier to pull out intended data all in one call rather than making multiple calls and getting data out in pieces.

```sql 
select * from Users; 
  create_date |             user_handle              | last_name | first _name 
--------------+--------------------------------------+-----------+-------------
  2018-06-06  | 2839f831-f82c-faj3-aof3-fj28ddks39ek | clark     | tyler  
  2019-02-01  | 6ab3b2d2-8e02-890c-bb6d-61a67cd43f31 | jones     | debbie    
  2010-01-10  | a0eebc99-9c0b-42f8-g3eh-6bb9bd380a11 | freemon   | mary  
  (3 rows)
```

In the grouping and lesson within this course, we wanted to pull out the row that had the earliest `create_date`. If we do `select min(create_date)`, and then pull out the `first_name`, we get an error. 

```sql 
select min(create_date), first_name from Users;
ERROR: column "users.first_name" must appear in the GROUP BY clause or be used in an aggregate function 
LINE 1: select min(create_date). first_name from Users;
```

Our database doesn't know how to handle the other columns while it's aggregating all the `create_date` columns down to one. If we add `first_name` to the `group by`, we see we get all three rows out, because now that `first_name` is grouped by unique first names, we'll get out the man per grouping of first names.

```sql 
select min(create_date), first_name from Users group by first_name;

     min     | first_name
 ------------+------------
  2018-06-06 | tyler 
  2019-02-01 | debbie 
  2019-01-10 | mary
(3 rows)
```

Instead, what we want to do is use a subquery. For that, we'll write, `select create_date, first_name from Users where create_date` equals a subquery, selecting out the `(select min(create_date) from Users);` table. 

```sql 
select create_date, first_name from Users where create_date = (select min(create_date) from Users);

     min     | first_name
 ------------+------------
  2018-06-06 | tyler 
(1 row)
```

You see that this returns my name for `first_name` and my `create_date`. In order to use a subquery, it needs to be wrapped in parenthesis. Our subquery here returns one man `create_date` from our `Users` table.

That's what we use to filter by, where our `min` from the inner query matches the `create_date` from the outer query. With this filter matching on the correct row, we can just pull out all the columns then. Subqueries can be used in other commands such as `insert`, `update`, and `delete`. Let's update the `first_name` column of the row that has the earliest `create_date` to `'danny'`. 

```sql
update Users set first_name = 'danny' where create_date = (select min(create_date) from Users);
UPDATE 1
```
```sql
select * from Users;
  create_date |             user_handle              | last_name | first _name 
--------------+--------------------------------------+-----------+------------- 
  2019-02-01  | 6ab3b2d2-8e02-890c-bb6d-61a67cd43f31 | jones     | debbie    
  2010-01-10  | a0eebc99-9c0b-42f8-g3eh-6bb9bd380a11 | freemon   | mary  
  2018-06-06  | 2839f831-f82c-faj3-aof3-fj28ddks39ek | clark     | danny 
  (3 rows)
```

Here you'll see that we've changed from Tyler to Danny. Not only can subqueries be used as a way to filter, but we can use them as a table to join on. Let's write, `select total, first_name from Users us inner join` on a subquery. We'll do `(select count(user_handle) as total, user_handle from Purchases group by user_handle)`, and then we'll say where user handles match, `p.user_handle = us.user_handle;`.

```sql
select total, first_name from Users us inner join (select count(user_handle) as total, user_handle from Purchases group by user_handle) p on p.user_handle = us.user_handle;
   total     | first_name
 ------------+------------
         1   | danny 
         2   | mary 
(2 rows)
```

With this in place, we'll see that Danny had one purchase and Mary had two purchases from the `Purchases` table. Let's walk through this again. The top-level query is going to give us one column that exists on users, and the other that we create from our subquery. Our subquery is getting a count of group by user handles.

This count tells us how many rows are in each group `user_handle`. We `inner join` on the `user_handle`. We pull out from the subquery and the user handles that already exist on the `Users` table. The returning results show us how many orders each person purchased.

Another option is using subqueries as a column value. For example, I want to return the average order size along with all of my `sku`'s and user handles inside of my `Purchases` table. I can add a subquery next to my other columns. 

```sql
select user_handle, sku, (select avg(quantity) from Purchases) from Purchases;
              user_handle             |                sku                    |       avg
--------------------------------------+---------------------------------------+--------------- 
 6ab3b2d2-8e02-890c-bb6d-61a67cd43f31 | 2839f831-f82c-faj3-aof3-fj28ddks39ek  | 1.50000000000
 a0eebc99-9c0b-42f8-g3eh-6bb9bd380a11 | a0eebc99-9c0b-42f8-bb6d-6bb9bd380a11  | 1.50000000000
 2839f831-f82c-faj3-aof3-fj28ddks39ek | a0eebc99-9c0b-42f8-bb6d-6bb9bd380a11  | 1.50000000000
 a0eebc99-9c0b-42f8-g3eh-6bb9bd380a11 | 2839f831-f82c-faj3-aof3-fj28ddks39ek  | 1.50000000000
(4 rows)
```

When I run this, I get all of my `user_handle`'s, all of my `sku`'s, and a consistent average across all of the quantities within the `Purchases` table.

To clarify, this isn't stating that the average for this user for the `sku` is this value. This is just the average quantity for all of the purchases made inside of our `Purchases` table. I will, though, post the solution for making this average be a reflection of this user handle and the skew in the notes.

Before you look, see if you can do it yourself. You'll want to enter another row with the same `user_handle` and `sku` and a different quantity to make sure it's working like it should.
