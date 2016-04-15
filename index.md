## Benefits of abstraction layers over sql

Database management

- Pooling/sharing/closing connections
- Automatic database selection
- Avoid vendor lock-in by creating a cross-backend abstraction
- Migrations - backward and forward

I don't even know how to safely manage connections, since django took care of that for me.

Query creation

- Automatic cleansing
- Automatic handling of various data types
- Relation abstraction, including hiding hinge tables
- Simpler interface - instead of string handling, chained method calls

We don't need database management, or at least it would be a pain to refactor to. We could use the query creation features, though.

## Problems with ORMs

The famous [object-relational impedance mismatch](https://en.wikipedia.org/wiki/Object-relational_impedance_mismatch)

- Data Types: ORMs map language semantics to database semantics. This is particularly a problem with references - hence relations being badly supported.
- Validation: many ORMs perform validation client-side, rather than respecting the database's semantics
- Transactions and data integrity: because of the problems with semantics and validation, sometimes the database's transactional model is ignored or re-implemented client side.
- Optimization: object graphs, duplicate validation, in-memory aggregation, multiple queries to substitute for transactions.

The more complex the use case, the harder it is to support.
    - Hinge tables with intermediate values are often supported, but confusing to configure
    - Complex aggregations and optimizations are often unsupported

## 2 Kinds of ORMs

- Object-oriented ORMs modify the data model to be useful in OO code (Django)
    - This has more impedance mismatch problems, but is very powerful for common use cases
- Query Building ORMs respect the data model while making it easier to work with. (Stingle)
    - This has fewer impedance mismatch problems, but isn't as powerful

My query builder is not an ORM. It's a DSL builder

## DSLs

Our current approach to MySQL has a number of steps. We:

1. Cleanse inputs
2. Format inputs
3. Concatenate inputs into phrases
4. Concatenate phrases into queries
5. Execute the query
6. Handle errors
7. Format returned data

MySQL is a database, including a transactional model, data semantics, error handling, and more. Steps 1-4 are just string building - there is very little concern for MySQL's richer semantics (string vs int, and sometimes dates). A SQL query is a string-based DSL.

An ORM abstracts away the DSL. A DSL builder abstracts the building of the string DSL.

Examples of sql building, compare to ORM

## Benefits

- This has no impedance mismatch beyond what we already deal with. 
- It's unopinionated, and can be used as much or as little as necessary.
- Used correctly, it validates that inputs are either trusted or cleansed
- Forces the user to handle certain edge cases regarding type and empty values
- Is number agnostic in most situations
- Shortcuts for cleansing, formatting, and concatenating phrases together

Programmers are lazy. This helps programmers be more careful about cleansing inputs and handling edge cases by making it convenient to cleanse, format, and concatenate inputs.

## Data Model

Walk through the code:

- Wrapper
- combine
- Value fundamentals
- Phrase
- combining phrases
- Configuration options for Value and Phrase

## Flexibility

It supports cleanse/quote/delimit based DSLs as well

- Url Query Strings
- CSV
- Terminal output

The code favors SQL as the primary supported abstraction, but if it seems valuable, I could put some more effort into making it DSL agnostic and extending it with a sql wrapper.

## Bonus: Databases

I'll compare a number of different databases, describing their strengths, weaknesses, and possible use cases at a high level. When I did this research a few weeks ago for my own purposes, I was amazed at the diversity of options, and how liberating it was to realize that data need not be rectangular. I'll touch on:

- Mysql
- Graph databases (neo4j, OrientDB)
- KV stores (Redis)
- Nonrel (Mongo)
- RDF
- Event Stores
- Datomic

Datomic will be the high point, of course. I'll talk about its many benefits - solid time model, distributed architecture, immutable data, attribute-oriented schemas, queries as data structures, writing custom procedures in java (or a compile-to-java language), and how it learned from sql, graphs, RDF, and Event Stores.