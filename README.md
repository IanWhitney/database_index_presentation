# Database Indexes<br />for Fun and Speed

> Ian Whitney
> whit0694@umn.edu
> @ian_whitney (Slack)


---

# Imagine An Unsorted Phonebook

| Row | Last Name | First Name | Middle Initial |
| --- | --- | ---- | ---- |
| 1 | Miller | Jessica | E |
| 2 | Grant | Anna | C |
| 3 | White | Beryl | D |
| 4 | Williams | Steven | R |
| 5 | Grant | Charlotte | A |
| 6 | Whitney | Ian | M |
| 7 | White | Arthur | D |

---

# Let's query that Phonebook 

```sql
select * from phone_book where last_name = 'Whitney'
```

^The database has no way of knowing where these records are.

^So all it can do is pull the entire table from disk into memory and check each row.

---

#  Let's add an index

```sql
create index phone_book_last_name on phone_book(last_name);
```

| Last Name | Row |
| ---- | ---- |
| Grant | 2 |
| Grant | 5 |
| Miller | 1 |
| White | 3 |
| White | 7 |
| Whitney | 6 | 
| Williams | 4 |

^An index creates a Database structure that orders the data by the indexed field

^It's now easy for the database to find all rows with a last name of Whitney.

---

# But Looking By Last Name Sucks

```sql
select * from phone_book where last_name = 'Whitney' and first_name = 'Ian';
```

^The database has to pull all Whitneys from disk and then find the ones that match by first name.

---

# We could add another index

```sql
create index phone_book_first_name on phone_book(first_name);
```

| First Name | Row |
| ---- | ---- |
| Anna | 2 |
| Arthur | 7 |
| Beryl | 3 |
| Charlotte | 5 |
| Ian | 6 |
| Jessica | 1 |
| Steven | 4 |

---

# This is probably better
## But...

- Your database now has to maintain two indexes
- And none of your users ever query by just first name
  - It's always last name & first name

---

# Let's add a better index

```sql
create index phone_book_first_last_name on phone_book(first_name, last_name);
```

| Last Name | First Name | Row |
| --- | ---- | ---- |
| Grant | Anna | 2 |
| Grant | Charlotte | 5 |
| Miller | Jessica | 1 |
| White | Arthur | 7 |
| White | Beryl | 3 |
| Whitney | Ian | 6 |
| Williams | Steven | 4 |

---

# Which Index Gets Used?

We now have 3 indexes. Which of them get used?

---

# Query by last_name, first_name

```sql
select * from phone_book where last_name = 'Whitney' and first_name = 'Ian';
```

Uses: `phone_book_first_last_name`

^Nothing surprising here. The index we created for this kind of query is used by this query

---

# Query just by first name

```sql
select * from phone_book where first_name = 'Ian';
```

Uses: `phone_book_first_name`

^This would use our first_name index. But no one does this. So delete the index. It's just taking up space and slowing things down.

---

# Query just by last name

```sql
select * from phone_book where last_name = 'Whitney'
```

Uses: `phone_book_first_last_name`

^The last/first name index offers the same sort as our last name index.

^It's sorted by last name first, so it's easy for the DB to identify what rows to pull from disk.

---

# The Rule

If the column(s) in your `where` clause are the left-most columns in a multi-column index: 👍

```sql
create index phone_book_last_first_name on phone_book(last_name, first_name);
```

✅ `where last_name = 'Whitney'`
✅ `where last_name = 'Whitney' and first_name = 'Ian'`
❌ `where first_name = 'Ian'`

---

# Delete that last name index!
## It's redundant

Slides:

> https://z.umn.edu/indexes

Video that taught me a bunch: 

> https://z.umn.edu/index_talk

