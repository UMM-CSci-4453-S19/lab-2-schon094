# Lab-2

## Outline
  - [Overview](#overview)
  - [A few SQL statements](#a-few-sql-statements)
  - [In class mini-lecture](#in-class-mini-lecture)
  - [The `SELECT` statement](#the-select-statement)
    - [Creating the table](#creating-the-table)
    - [Giving it some data](#giving-it-some-data)
      - [Use `INSERT`](#use-insert)
      - [Basic form of `SELECT`](#basic-form-of-select)
      - [Renaming a column](#renaming-a-column)
      - [Getting fancy with columns](#getting-fancy-with-columns)
      - [The `WHERE` clause](#the-where-clause)
      - [Logicals in the `WHERE` clause](#logicals-in-the-where-clause)
      - [The `LIMIT` clause](#the-limit-clause)
      - [The keyword `DISTINCT`](#the-keyword-distinct)
      - [The REAL power:  **JOINS**](#the-real-power--joins)
      - [The `EXPLAIN` keyword](#the-explain-keyword)
  - [Put together a small group](#put-together-a-small-group)
  - [An SQL tutorial](#an-sql-tutorial)
  - [Checklist of what to do](#checklist-of-what-to-do)
  - [References](#references)

## Overview

The goal of this lab is to get you used to working with tables. In particular we are going to spend some
time working with queries (the results of a `SELECT` statement). New commands that you will be using
are:

* `UPDATE`,
* `DELETE`, and
* `SELECT`.

(You might not end up needing to use `DELETE` today, but it's good to know in case you make any mistakes)

In today's lab.  I want everybody to work separately.  You can ask each other questions, but it is important that everybody does their own work today.  (it is fine to work in groups-- but **each** of you needs to do everything in the lab).  In addition, I want the commands to be **typed**.  Copy-and-paste is quick, but it is not a good way to learn.

## A few SQL statements

Below are a few SQL statements and their structure.  Any expression enclosed in angle-brackets is **meant to be replaced**.

```
	UPDATE <table> SET <field>="<value>" WHERE <condition>;
	INSERT INTO <table> (<col list>) VALUES (<value list>);
	DELETE FROM <table> WHERE <condition>;
```

The [references](#references) section contains a link to *exhaustive* information about the 1999 SQL standard (not all of which works with MariaDB).  Also the quotes on `UPDATE` are only needed when the value is a string.

Here are a few more SQL commands:

```
	CREATE TABLE <name> (<column definitions) <options>;
	DROP TABLE <name;
	TRUNCATE TABLE <table>; 
```

The `TRUNCATE TABLE` command will remove all the data in a table without changing its structure (this is **very** useful)

SQL commands are broken into **clauses**.  For example, the command `UPDATE inventory SET unit=NULL WHERE unit="patty";` has  a `SET` clause and a `WHERE` clause.  This will make a lot more sense after you work through the examples provided below.  It is important to note that the **order** of the clauses is important and many SQL statements will **fail** if the clauses appear in the wrong order.

## In class mini-lecture

The theoretical basis behind data storage in RDBMS' are sets (in the mathematical sense).  For those unfamiliar with the idea (or if you just want a refresher), I'll give a mini-lecture at the whiteboard discussing:

* Mathematical idea of a **set**
* Some common **set operations**
   * union (join)
   * intersection (meet)
   * set differences
   * Cartesian product
 * The idea of a **universe**
 * The mathematical definition of a **relation**
 * Ordered sets of data as **tuples**
 * Relating these ideas to RDBMS
   * Table as a relation
   * records (rows) as elements 
   * The need for unique identifies (KEYS and PRIMARY KEYS)

Whether you listen to this or not, you should read the information at <http://rdbms.opengrass.net/2_Database%20Design/2.1_TermsOfReference/2.1.2_Keys.html>.

## The `SELECT` statement

We're going to start with `SELECT`. But in order to do that we're going to need to make a few tables to
play with. 

Our eventual goal is to make a *point of sales* system, so we are going to start in a fairly simple fashion:

### Creating the table

1. Log in using your username
2. Be sure that your database is the current one
3. Create a table named `inventory` with the following fields (read the **full** question before attempting any SQL)
   * `item`
   * `unit`
   * `amount`
   * `id`
  
I want you to use the following description for the `id` field:

```
id INTEGER PRIMARY KEY AUTO_INCREMENT NOT NULL
```

Notice that the format is similar for any field:  `<col name> <col type> <extras>`.  The column definitions (the other name for fields) are separated by commas.

From the link provided above you should recall that a **PRIMARY KEY** is an important (subjective) collection of attributes that 

* has a unique value for each row (super key)
* is no longer a super key if any attribute is dropped

In the column definition I provided above, you have told the DBMS that the column `id` serves as the primary key for the table.  The extra `AUTO_INCREMENT` ensures that this field (see how I've used all 3 of the words?) has a unique value for each row, and `NOT NULL` ensures that there is a value in the field… hence it really is a primary key.  (DBMS allow people to cheat a bit-- technically a column can't serve as a PRIMARY KEY if it has a NULL value)

### Giving it some data

#### Use `INSERT`
Now that the table exists, use `INSERT` to add some data:

Insert 5 or 6 items of varying amounts and values into `inventory`.  You'll want at least a few low numbers and at least a few high numbers in the `amount` field.   (we'll use this later).<BR><BR>Add another item and *"accidentally"* give it the same name as something already in the table, but make certain that at least one of the fields differ between the two records.  (The up-arrow is your friend for manual entries like these)

#### Basic form of `SELECT`
The basic form of the `SELECT` command is `SELECT <cols> FROM <table>;`.  Experiment with that.  For example:

```
		SELECT item,unit FROM inventory;
		SELECT unit FROM inventory;
		SELECT unit, unit, item FROM inventory;
```

The `*` option will show **all** columns/fields:

```
		SELECT *  FROM inventory;
		SELECT *, item FROM inventory;
```

Notice that it is perfectly ok to repeat a column in the `SELECT` statement, and that you control the order of the columns (unless you use `*`).

#### Renaming a column
You can rename a column using the `AS <new name>` clause (note the use of quotes and the absence of quotes):

```
		SELECT item AS 'mega stuff' FROM inventory;
		SELECT item AS stuff, amount, unit AS 'counting thing' FROM inventory;
```
#### Getting fancy with columns
SQL has a fairly large collection of functions.  Some are standard (based upon WHICH version of SQL you are using), but just about everybody has their own collection... just a bit below is MariaDB's functions and operators.  Pay particular attention to the comparison operators and how they deal with NULL.  Also notice that some functions show up as SQL keywords (look at BETWEEN).  After the link I will provide a few comments on what to ignore or skim.  Pay attention to the rest:

<https://mariadb.com/kb/en/functions-and-operators/>

Here are a few things to keep in mind:
* Ignore the Dynamic Columns functions for now-- they're a way for us to get around some of the limitations of relational database and to let us interface with a NOSQL database like Cassandra.  (I'm hoping to get back to them near the end of the course).
* For the moment, the function that are used with GROUP BY will remain mysterious (we'll fix that soon).
* We're not going to go into the GEOMETRY and GEOGRAPHY functions (but they might make a good group project... hint hint)
* Skim over the Miscellaneous Functions
* Ignore anything about user-defined functions, spider functions, and analyze (for now).

NOTE:  hidden in the string functions (to which you WANT to pay attention) are the functions that deal with regular expressions.  This is useful-- but not as powerful as some regular expression engines (ask me if you don't know what a regular expression is).  For reference here's the direct link:  https://mariadb.com/kb/en/regular-expressions-overview/ 

It might not be clear from the documentation (unless you look very closely), but things like `&&` and `AND` are synonyms. 

The functions can be used in column list and can use columns as arguments:

```
SELECT left(item,3) AS leftiest FROM inventory;
```

You can combine this with the * option to add your column (but be careful, I'm pretty sure that `*` has to come first):

```
SELECT *, left(item,3) AS leftiest FROM inventory;
```

You do not, however, have to use columns as arguments:

```
SELECT "peanut butter jelly" AS time, item, unit FROM inventory; 
```

If you leave out the new column name then the column will be named after the expression:

```
SELECT left(item,3) AS leftiest FROM inventory; 
SELECT left(item,3) FROM inventory; 
```

#### The `WHERE` clause

The `WHERE` clause acts as a filter.  It only displays rows that meet the criterion:

```
SELECT * FROM inventory WHERE amount < 50;
```

Think of the condition as being applied to each row one at a time and only when a TRUE is returned is that row allowed into the output.  This will return all rows that contain an n:

```
SELECT * FROM inventory WHERE item REGEXP 'n';
```
OR
```
SELECT * FROM inventory WHERE item RLIKE "n";
```

Note.  It's not case sensitive.
	
If I wanted all rows where item STARTED with n:
```
SELECT * FROM inventory WHERE item REGEXP '^n';
```
OR:
```
SELECT * FROM inventory WHERE item LIKE "n%";
```	

Remember this was originally designed for business, so there is a "sort-of" approximate match called `SOUNDS LIKE` which takes the English pronunciation into consideration.
	
For example, I have nacho cheese as one of my items, and this query (the results of a select statement) works to find them:

```
SELECT * FROM inventory WHERE item SOUNDS LIKE "nawch";
```		

#### Logicals in the `WHERE` clause
Although our sample table is still pretty small see if you can do a search for everything whose name contains a particular letter and has an amount less than 50 (refer to the first link above if necessary)

#### The `LIMIT` clause

Add `LIMIT <num>` at the end of a select to control the max number of rows:

```
SELECT * FROM inventory LIMIT 3;
```

This is particularly useful when you are first crafting a query because it can be pretty easy to mess up and produce a query that returns thousands of rows.

#### The keyword `DISTINCT`

The `SELECT` command has a keyword called `DISTINCT`.  Placement is important-- it has to be immediately after SELECT.  Give it a try:

```
SELECT DISTINCT item FROM inventory;
SELECT DISTINCT id FROM inventory;
```

#### The REAL power:  **JOINS**

Before proceeding **make a second table** in your database called `prices`.  The price table should have columns named

field  | data-type
-------|---------
prices | fixed with two decimal places
notes  | text
id     | Same as in `inventory`, but this time do NOT make it `AUTO_INCREMENT` or the `PRIMARY KEY`

After you have made the table **add a few entries**.  Just pick a few things.  

ProTip:  In an `INSERT` you can leave out the initial parenthetical clause that lists the columns if you are adding values for every column.  For example, here is a description of my prices table:

```
mysql> describe prices;
	+-------+--------------+------+-----+---------+-------+
	| Field | Type         | Null | Key | Default | Extra |
	+-------+--------------+------+-----+---------+-------+
	| id    | int(11)      | NO   |     | NULL    |       |
	| price | decimal(5,2) | YES  |     | NULL    |       |
	| notes | text         | YES  |     | NULL    |       |
	+-------+--------------+------+-----+---------+-------+
	3 rows in set (0.00 sec)
```

And here's an `INSERT` into that table:

```
INSERT INTO prices VALUES (1,12.43,"Buy from Ritchie");
```

Here's what mine looked like (your mileage may vary):

```
	mysql> select * from prices;
	+-----+--------+------------------+
	| id  | price  | notes            |
	+-----+--------+------------------+
	|   1 |  12.43 | Buy from Ritchie |
	|   2 | 100.19 | Versoin A        |
	|   3 | 200.19 | Premium          |
	| 100 |  19.18 | Chicken Flavored |
	+-----+--------+------------------+
	4 rows in set (0.00 sec)

```

Oh... wait.... it looks as if I made a mistake.  Let's fix `Versoin A` and change it to the proper `Version A`:

```
UPDATE prices SET notes="Version A" WHERE id =2;
```

(be careful-- you can accidently change ALL the values in a particular field if you are not careful.		

So, back to it:

```
	mysql> select * from prices;
	+-----+--------+------------------+
	| id  | price  | notes            |
	+-----+--------+------------------+
	|   1 |  12.43 | Buy from Ritchie |
	|   2 | 100.19 | Version A        |
	|   3 | 200.19 | Premium          |
	| 100 |  19.18 | Chicken Flavored |
	+-----+--------+------------------+
	4 rows in set (0.00 sec)
			
	mysql> select * from inventory;
	+---------------+---------+--------+----+
	| item          | unit    | amount | id |
	+---------------+---------+--------+----+
	| Buffalo chips | patty   |   1200 |  1 |
	| nacho cheese  | gallons |    100 |  2 |
	| nacho cheese  | gallons |     20 |  3 |
	| popcorn       | gallons |      5 |  4 |
	| hot dogs      | units   |    120 |  5 |
	| buns          | units   |      3 |  6 |
	+---------------+---------+--------+----+
	6 rows in set (0.00 sec)
```

The important thing is that both tables have entries not found in the other and that neither table has too many entries (if the product of the number of rows on the two tables is too large then this next step is going to be unpleasant).
		
So... it's go time!  Give this a shot:

```
SELECT * FROM inventory,prices;
```

To verify that the order of tables in the FROM clause matter try this:
```
SELECT * FROM prices,inventory;
```

Now try this one:
```
SELECT * FROM inventory, inventory;
```

That one should give an error.... but this one won't:
```
SELECT * FROM inventory, inventory AS b;
```

The key is that you have given the second table an alias (more on this soon).   Type this one again:

```
SELECT * FROM inventory, prices;
```

Be sure to look carefully at the `item` column.  Now try this:
```
SELECT item FROM inventory, prices;
```

and try this:
```
SELECT item,amount FROM inventory, prices;
```

I hope it's clear what's happening.  The `SELECT *` statement with TWO tables takes every row in the first table, and every row in the second table and concatenates them together into a bigger row in every possible combination.  (Math people-- this is an example of a Cartesian Product).
		
If the first table has 10 rows, and the second table has 3 rows, then the JOIN (that's a technical term) has 30 rows.  
		
There ARE a few limitations in select columns... Try this:
```
SELECT id FROM inventory, prices;
```

Notice the  problem is the ambiguity.  Fix it like this:
```
SELECT inventory.id FROM inventory,prices;
```

and 
```
SELECT prices.id FROM inventory,prices;
```

Notice that the values are different (although there are still the same number of entries.)
```
SELECT inventory.id FROM inventory,prices;
SELECT inventory.id FROM prices,inventory;
```

(at least in my case) gives the same results... this actually surprises me, and I am pretty certain it has to do with MariaDB's attempt to optimize queries (another name for the output from a `SELECT` statement).
		
That still doesn't help us fix
```
SELECT id FROM inventory, inventory as B;
```

or does it?  Try this:
```
SELECT B.id FROM inventory, inventory as B;
```

Or this:
```
SELECT bob.id  FROM inventory AS bob, inventory as doug;
SELECT doug.id  FROM inventory AS bob, inventory as doug;
```		

And now you are ready... try this:
```
SELECT * FROM inventory, prices WHERE inventory.id=prices.id;
```

See the power?  You can bind these tables together in a way that allows you to extract everything you need.  There are other ways to do the same thing:
```
SELECT * FROM inventory INNER JOIN prices ON inventory.id=prices.id;
SELECT * FROM inventory CROSS JOIN prices ON inventory.id=prices.id;
```

Notice that before we used the WHERE clause to control which rows to include, now we're using the ON clause in collaboration with the INNER JOIN or CROSS JOIN (they're synonyms) commands.  Functionally this works the same.
		
What happens if want to link the tables on the id field, but still want to include data from inventory when there isn't a matching ID value in prices?  Try these (pay careful attention to where the NULL's occur):
```
SELECT * FROM inventory LEFT JOIN prices ON inventory.id=prices.id;
SELECT * FROM inventory RIGHT JOIN prices ON inventory.id=prices.id;
SELECT * FROM prices LEFT JOIN inventory ON inventory.id=prices.id;
SELECT * FROM prices RIGHT JOIN inventory ON inventory.id=prices.id;
```

You can use this technique when combined with a `WHERE` clause to find rows that have a value in one table, but not a corresponding one  in  the other:
```
SELECT * FROM inventory LEFT JOIN prices ON inventory.id=prices.id where prices.id IS NULL;
```

Of course, every  `LEFT JOIN` has an equivalent `RIGHT JOIN`.
		
There is also (theoretically), an `OUTER JOIN` which, effectively is a combination of the `LEFT JOIN` and the `RIGHT JOIN` (in the way you would expect), however, MariaDB doesn't seem to implement it correctly.

Before continuing **read the following explanation of SQL joins.  Be sure to do as many of the examples as you can-- in particular create the tables**:   http://www.codinghorror.com/blog/2007/10/a-visual-explanation-of-sql-joins.html (but remember that mariaDB doesn't do `OUTER JOINS` correctly... so some of this knowledge is, for the moment, theoretical).

#### The `EXPLAIN` keyword
By putting the keyword `EXPLAIN` (or `DESCRIBE`-- they're synonyms) in front of a query you get back a table that tells you something about how mariaDB is attempting to optimize your query:

```
EXPLAIN SELECT * FROM inventory;
EXPLAIN SELECT * FROM prices;
EXPLAIN SELECT *,"bob" AS yeti FROM prices;
EXPLAIN SELECT * FROM inventory, prices where inventory.id=prices.id;
EXPLAIN EXTENDED SELECT * FROM inventory, prices where inventory.id=prices.id;
EXPLAIN SELECT * FROM prices RIGHT JOIN inventory ON inventory.id=prices.id;
```

Here's a link for more information… but we still need some background before filling in all the pieces:
    https://mariadb.com/kb/en/explain/

## Put together a small group

We are going to be putting together a sample concession stand/ticket sales application.  Try to figure out who you want to be in your group and spend some time brainstorming the tables and types of interactions that would be necessary.  

Don't invest too heavily in the design.  We will be learning some new technique soon (in particular, we are going to be discussing **data normalization**).  Email me your group name with the subject `cs4453-f17-groups`.

## An SQL tutorial

Do this tutorial for reinforcement:  http://www.sqlcourse.com/intro.html


## Checklist of what to do

* Type in the commands I outlined above while reading through the lab (each person should do this)
* Create the `inventory` table
    * Fields should be item, unit, amount, id (and of reasonable data-types)
    * `id` should be AUTO_INCREMENT, etc
    * There should be 5 or 6 entries in the table
* Create the `prices` table
   * Fields should be `prices`, `notes`, `id`
   * data-types of fields should be as indicated in the lab
   * There should be at least 4 entries in the table
* Do the examples in the [join tutorial](http://www.codinghorror.com/blog/2007/10/a-visual-explanation-of-sql-joins.html)
* Do the [sqlcourse tutorial](http://www.sqlcourse.com/intro.html)
* Do some brainstorming **AS A GROUP**
   * Email me your group with the subject `cs4453-f17-groups`.
   * Add a file to your Lab 2 repository named `labs2.txt`:
      * indicating group members
      * indicating your thoughts about the data and user interactions with the data

## References

* [SQL 99 reference](https://mariadb.com/kb/en/sql-99/)
