---
title: Querying the data
---
{% include mathjax.html %}

This is the third (draft) chapter of a new book on relational databases (using Postgres) that I'm working on as a side project.  Stay tuned for additional chapters.  The book under development can also be viewed [at Leanpub](https://leanpub.com/relating-to-the-database), which supports commenting, and also will allow me to bundle the book with video lectures.  I appreciate your feedback!

Sorry the tables and LaTeX equations don't work in Svbtle... check out the e-book to see how they're supposed to look.

## A declarative query language

You have already seen some of the **Structured Query Language (SQL)** which is used to express queries in Postgres (and every other relational database that I know of) and you're going to see a lot more in this book's chapters.  You have "programmed" several queries but here's one thing you may not know: SQL is not a programming language.  A computer program written in a language like Python, Java, or C++ is *imperative*---it gives a computer a sequence of instructions to carry out until it finished.  SQL, by contrast, is a **declarative language**.  In SQL queries, you describe the result that you want, not *how* the computer should obtain it.  That turned out to be a genius move by the creators of the first relational databases.

Inside a DBMS like Postgres is a special function called the **query optimizer** which processes a SQL query and generates an **execution plan** for how best to obtain the desired result.  In a complex query that incorporates multiple tables, there may be several steps in the plan, some slow and some quicker.  These operations may include **full table scans** (reading an entire table from disk; slow), **index scans** (much quicker), and different types of join operations (see Table 3-1).  Doing them in a certain order may be faster than doing them in another order, and this can make a big difference in a database with millions or billions of rows.  Because SQL is *declarative*, the query optimizer has the freedom to choose the most efficient sequence.

[Table 3-1: Sample of primitive operations in query execution]

| Operation | Meaning |
|--------------------|
| Full table scan |  Read every row in the table and find the one(s) specified by the query.  |
|--------------------|
| Index scan (aka "seek") |  Search an index to quickly find locations of the rows specified by the query.  A database index is conceptually like the index in the back of a book; it makes finding the right "page" much quicker, more so when the book is longer. |
|--------------------|
| Table access |  Go directly to the location of the specified row(s) and read the data. |
|--------------------|
| Hash join | A two-phase algorithm to quickly join two tables based on an equality condition.  |
|--------------------|
| Nested loop join | A slower join algorithm that accommodates inequality conditions and other unusual joins.  |


Here is a key point that I'll come back to repeatedly: you should take advantage of the work that the database developers have already done.  Yes, you *could* write your own execution plan, or your own program for processing the data, but it would take a lot of time and you might not get it right.  Database engines are designed by some of the smartest computer scientists in the world and honed by practical experience for years, and they have very likely anticipated queries like yours.  Give the database the freedom to optimize, and it will generally do an excellent job.

*ASIDE: My philosophy is at odds with received wisdom.  When I was first learning databases, I was often told that you should keep your business logic (i.e., your program) out of the database.  The SQL dialects of popular databases (Oracle, SQL Server, etc.) were very similar in their DML (**data manipulation language**) commands like `SELECT` and `INSERT`, but had very different implementations of views, triggers, stored procedures, and other programming features (all of which will be further discussed in Chapter 7).  The theory therefore was that if you used the latter, you'd be "locked in" to one database platform, so in order to keep your options open you should only use the database for the simplest DML operations.  These are known as the **CRUD operations**: create, read, update, and delete.*

I disagree.  First off, I don't think lock-in is a very worrisome problem, especially when your chosen database has a long track record of success and is not too expensive.  Second, the rewards of committing to PostgreSQL or any quality database platform are well worth the small risk that it might be more expensive to switch to another platform at some hypothetical point in the future.  These days, databases typically support not just one application, but several---perhaps a web page, two mobile apps, and a public API.  In that case, do you really want to write the code for simple tasks (such as authenticating a user's login) over and over again in each of the programming languages used?  Implementing parts of your program's logic in the database reduces redundancy, may improve security, and allows you to take advantage of features like the query optimizer to make your program more efficient. 

## Relational operations

Relations (remember, this is the mathematical term for what we're calling "tables") are sets of tuples, as discussed in Chapter 2.  There are a number of mathematical operations that can be performed on them, with the interesting property of **closure**: the result of each **relational operation** is itself a relation.  The clauses of a SQL query can be interpreted as a specification of relational operations to be performed on the specified tables.  Interestingly, just as you might simplify a complicated equation in high school algebra before solving it, the query optimizer might use **relational algebra** to build its execution plan---choosing which operations to perform first in order to reduce the amount of computation it will have to do to finish the job.

The key relational operations identified by E. F. Codd and derived from set theory are the **projection**, **selection**, and **Cartesian product** operations, but to this database developers have added several more very useful operations, particularly **extended projection**, aggregation, grouping, and sorting.  See Table 3-2.

[Table 3-2: Important relational operations in SQL queries]

| Operation | SQL clause | Symbol | Meaning |
|-----------|------------|--------|---------|
| Projection | `SELECT` | $$\sigma$$ | Return only the specified columns |
| Selection | `WHERE` | $$\Pi$$ | Return only the rows that match specified criteria |
| Cartesian product | `CROSS JOIN` | $$\times$$ | Return every combination of a row from table 1 with a row from table 2 |
| Natural join | `NATURAL JOIN` | $$\Join$$ | Return all combinations of rows in specified tables that are equal on their common column |
| Extended projection | `SELECT` | $$\sigma$$ | Generate new columns in the resulting table, such as the results of calculations or logical tests |
| Aliasing | optional `AS` | $$\rho$$ | Assign a (new) name to a column in the resulting table | 
| Aggregation | `SUM`, `COUNT`, `AVG`, etc. | $$G_{f(x)}$$ | Replace original rows with a single row containing the computed result |
| Grouping | `GROUP BY` | $$_xG$$ | In combination with aggregation, split the original data into subsets to yield subtotals, subaverages, or whatever |
| Sorting | `ORDER BY` | n/a | Re-arrange the rows in a specific order | 

### Basic relational operations from set theory

**Projection** is the operation of reducing a table to a subset of its columns, and in SQL it is expressed as a list of columns following the `SELECT` keyword, for example:

        SELECT name, age
        FROM players;

**Selection** is the operation of reducing a table to a subset of its rows, and in SQL it is expressed as a logical test (for equality or inequality) follwing the `WHERE` keyword.  Multiple conditions may be combined into one with the `AND` and `OR` keywords if needed.  For example:

        SELECT *
        FROM players
        WHERE team='Patriots' AND position='QB';

These are certainly the most common operations, and most queries will employ both.  Consider the query

        SELECT name, age
        FROM players
        WHERE team='Patriots' AND position='QB';

This query may be expressed in relational algebra as $$\Pi_{name,age}(\sigma_{team=Patriots \land position=QB}(players))$$.  This formulation implies that the selection operation should be computed first, and then the projection operation.  But because it is an algebra, and because the outcome of every operation is another relation, we could just as easily flip it around, i.e.: $$\sigma_{team=Patriots \land position=QB}(\Pi_{name,age}(players))$$.  This kind of flexibility gives the query opimizer room to make choices that speed up the query.  

The third of the "original" relational operations is the **Cartesian product** operation which joins every row of one table with every row of a second table.  The Cartesian product is expressed in PostgreSQL as `CROSS JOIN` and one way it sometimes comes in handy is to generate a cross-tabulation of the rows of two tables.  For example, if you want a report to yield some statistics about every football team in every year (perhaps to build a line graph?), the core of the query might be:

        SELECT * FROM teams CROSS JOIN seasons;

Or in relational algebra, $$teams\times seasons$$.  The more common type of join, as discussed in Chapter 2, is a **natural join**, where each row of one table is joined with only the rows of the other table that have matching values of a specific column (i.e., a foreign key - primary key relationship).  In Postgres there is actually a `NATURAL JOIN` keyword that works when the columns literally have the same name.  If they have different names (for example, if a "players" table has a FK called "team_id" but in the "teams" table it's simply called "id"), you can use either a `JOIN` clause or a `WHERE` condition to effect the join.  These are three ways you might perform a natural join on two tables in SQL:

        SELECT * FROM players NATURAL JOIN teams;
        SELECT * FROM players JOIN teams ON player.team_id=team.id;
        SELECT * FROM players, teams WHERE player.team_id=team.id;

In relational algebra, the natural join is expressed as $$players\Join _{team\_id=id} teams$$; the subscript expressing the join condition can be omitted if the FK-PK relationship is obvious.  You *could* perform a natural join by first taking the Cartesian product and then *selecting* the rows where the FK matches the PK, à la $$\sigma _{team\_id=id} (players \times teams)$$, and in theory this is what the database engine is doing.  In practice, the query optimizer will use an algorithm like a *hash join* to perform an equality join much more quickly.

**Inequality joins** are also possible.  If you want to join each player with teams he is *not* on, in order to perform some kind of comparison, you might do the following:

        SELECT * FROM players JOIN teams ON players.team_id != teams.id;

In relational algebra notation this is $$players\Join _{team\_id \neq id} teams$$.  Such a join is generally going to be quite expensive in computational terms because the database engine must perform a *nested loop*: for each row of the "players" table it must loop through the entire "teams" table to find relevant rows.

### Extensions to the relational toolkit

Although relational modeling and relational algebra originate in set theory, database developers and users have made numerous pragmatic extensions to the original theory-derived set of methods we can apply.  After all, a database isn't an academic exercise, but a practical business tool.

The idea behind **extended projection** is that a query can give us not only a selection of columns *from* the original table(s), but can also produce new columns as a result of calculations or logical tests.  For example:

        SELECT running_yards + passing_yards FROM game_results;

A closely associated idea is that of **aliasing** (also known as the "rename" operation), an operation that changes the name of a column or assigns a name to a column that doesn't have one (such as the calculated column above).  In Postgres, you can use the optional `AS` keyword, or simply provide an alias after specifying the column:

        SELECT running_yards + passing yards AS total_yards FROM game_results;
        SELECT first_name || ' ' || last_name AS full_name FROM players;
        SELECT age > 35 oldguy FROM players;

The last query above is an example that contains a logical test, "`age > 35`", and the result will be a column called "oldguy" that contains Boolean values: "true" and "false".  The `AS` keyword is omitted; it is optional, but may make your queries easier to read.  

Another powerful extension to relational algebra is **aggregation**; this allows us to generate new *rows* that do not come from the original tables, but instead are the result of calculations over some or all of the original rows.  The aggregations you will use most frequently are `SUM` and `COUNT`, but a number of other [aggregate functions in Postgres](https://www.postgresql.org/docs/current/static/functions-aggregate.html) are available, particularly for statistics such as `MIN`, `MAX`, `AVG` and so on.  

Aggregation works together with the operation of **grouping**, which identifies the set(s) of rows to be aggregated together.  If no grouping condition is set by a `GROUP BY` clause in the SQL, all rows are aggregated into one result.  To count the number of teams, for example, we could simply do:

        SELECT COUNT(*) FROM teams;

If we want to compute aggregates for subsets of the data, we use `GROUP BY` to generate a group for each distinct value of a particular column, for example:

    S   ELECT team, AVG(salary) FROM players GROUP BY team;

In relational algebra notation, the grouping and aggregation operations are denoted by a capital "G"; the grouping column in a preceding subscript and the aggregation function in the following subscript.  The above example would be expressed $$_{team} G _{AVG(salary)} (players)$$.

A final important operation is **sorting**.  Although in theory the order of rows in a relation is meaningless (it's a set), in practice we want to sort the rows into some kind of meaningful order.  There is no standard notation for this operation in relational algebra, but in SQL it's expressed in the `ORDER BY` clause of a query:

        SELECT * FROM players ORDER BY last_name, first_name;

The relational operations listed here are the key components from which you'll build most of your queries, and they are common to virtually all relational databases.  In Chapter 5, we'll introduce you to some additional relational patterns that are useful in special cases, and in Chapter 7 we'll explore some of the special features of PostgreSQL that distinguish it from other relational databases.

## Queries within queries

SQL queries can be much more complex than you have seen so far, when the answer to one query depends on the answer to others.  In such cases, a complete query called a **subquery** can be nested within another.  Subqueries are most often found in the `WHERE`, `FROM`, and `SELECT` clauses.  For example, this query names all players who earn a salary greater than the average:

        SELECT last_name, first_name, salary FROM players
        WHERE salary >
          (SELECT AVG(salary) FROM players);

The inner query, "`SELECT AVG(salary) FROM players`" is evaluated first and yields a single numerical result.  Then the outer query is evaluated to finish processing the query.  Just like in a mathematical expression, parentheses set the subquery apart from the clauses of the surrounding query.  (I also indented the subquery, but this is not necessary; PostgreSQL like most databases is indifferent to whitespace like spaces, tabs, and line breaks.)

Subqueries in the `FROM` clause, when evaluated, produce result sets that act like tables in the outer query.  To identify all the quarterbacks in the AFC East, we could join the "players" table with the result of a subquery that selects all teams in that division:

        SELECT last_name
        FROM players JOIN 
          (SELECT * FROM teams WHERE conference='AFC' AND division='East') AS afceast_teams
          ON players.team=afceast_teams.team_id
        WHERE position='QB';

In this example you saw that the `AS` keyword can be used to assign a name to a table, just as previously we saw it used to assign a name to a column.  When using a subquery as a table in the `FROM` clause, it *must* be given a name.  The `AS` keyword is optional, though I think it makes the query easier to read.

Another place you will often see a subquery is the `SELECT` clause.  Such a query will almost always yield a single numeric result, because in the `SELECT` clause it generates a value for a single column of query output.  Here is a simple example that lists the total salary budget for each team.  (Note that this could also be accomplished using a join and a `GROUP BY`... there are more ways than one to solve many SQL problems.)

        SELECT city, team_name,
          (SELECT SUM(salary) FROM players WHERE players.team=teams.team_id) AS total_payroll
        FROM teams;

This is a case of a **correlated subquery**, meaning that this subquery depends on some value from the outer query (namely `teams.team_id`).  Therefore, the subquery will be executed numerous times, once for each row of the "teams" table.  The queries in the previous two examples are noncorrelated (or uncorrelated) subqueries which need only be executed once.  Correlated subqueries are potentially much slower, so it could be preferable to solve this problem with a join instead of a subquery, but that's not possible in every case.  My philosophy is to trust the query optimizer (and its developers) to find the fastest way to execute the query, and not over-think the SQL: if it ain't broke, don't fix it.

Finally, subqueries can be nested within subqueries.  Indeed, there may be several levels of such nesting, making for some pretty complicated queries.  As you solved complicated equations in high school by first "simplifying the expression", the query optimizer may have many ways to simplify and sequence things behind the scenes so that the query's result can be obtained as efficiently as possible.
