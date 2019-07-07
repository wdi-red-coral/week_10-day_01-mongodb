[![General Assembly Logo](https://camo.githubusercontent.com/1a91b05b8f4d44b5bbfb83abac2b0996d8e26c92/687474703a2f2f692e696d6775722e636f6d2f6b6538555354712e706e67)](https://generalassemb.ly/education/web-development-immersive)

# An Introduction to MongoDB

## Objectives

By the end of this, developers should be able to:

- Interact with MongoDB databases and collections using the MongoDB shell.
- Create, Read, Update, and Delete documents in MongoDB collections using the
  MongoDB shell.

## Preparation

1. Fork and clone this repository
1. Create a new branch, `training`, for your work.
1. Checkout to the `training` branch.
1. Install dependencies with `npm install`.

## Introduction

Relational databases are good at modeling data that fits into tables.  What do
you use if your data isn't that structured?

Perhaps a [NoSQL](https://en.wikipedia.org/wiki/NoSQL) data-store.

MongoDB is a schema-less document-store that organizes documents in collections.
What does this mean?

### Terminology

| **Relational Database Term** | **MongoDB Term** |
|:-----------------------------|:-----------------|
| database                     | database         |
| table                        | collection       |
| row                          | document         |
| column                       | field            |

## Installation

- Mac https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/
```
  brew install mongodb

  brew services restart mongodb
```

- Ubuntu https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/
  
- Windows https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/

## Create a Database

### Code Along

We'll use `mongo-crud` as the database to hold our tables and
[mongo](https://docs.mongodb.org/manual/reference/program/mongo/) to interact
with it.  `mongo` is MongoDB's command line client which lets us execute
commands interactively and from scripts.

First let's fire up our server:

- Mac OS only:

```bash
brew services start mongodb
```

- Ubuntu Only:

```bash
sudo service mongod start
```

- WSL Only:

```bash
sudo service mongodb start
```

The command to enter into the MongoDB shell is `mongo <name-of-database>`:

```bash
$ mongo mongo-crud
MongoDB shell version v3.x.x
connecting to: mongo-crud
>
```

The command to display the current database in use is `db`:

```bash
> db
mongo-crud
>
```

The command to list databases is `show databases` (or `show dbs`):

```bash
> show databases
local  0.000GB # or local  0.078GB
>
```

Unlike PostgreSQL, MongoDB lets us select a database that hasn't been created.
When we add a collection, the database will be created.

For instance, if we didn't specify the database on the command line, we can
connect to a database with `use <database-name>`:

```bash
$ mongo
MongoDB shell version v3.x.x
connecting to: test
> db
test
> use mongo-crud
switched to db mongo-crud
> db
mongo-crud
> show databases
local  0.000GB # or local  0.078GB
>
```

## Importing Data

To help us practice interacting with a Mongo database, we'll want some data to
work with. MongoDB's [`mongoimport`](https://docs.mongodb.org/manual/reference/program/mongoimport/) command will let us load bulk data from a
`JSON` or `CSV` file.

Our first collection will be called `people`. It has no entries.

```bash
> show collections
> db.people.count();
0
```

This is a common pattern in MongoDB: you can refer to things that don't yet
exist, and it will cooperate.  MongoDB won't create them until you give it
something to remember.

### Demo: Bulk Load Books

Watch as I load data in bulk from `data/books.csv`.  We'll save the
command in `practice-scripts/import/books.sh`.

```bash
mongoimport --db=mongo-crud --collection=books --type=csv --headerline --file=data/books.csv
```

### Code Along: Bulk Load People

First, we'll load data in bulk from `data/people.csv`.  We'll save the
command in `practice-scripts/import/people.sh`.

```bash
mongoimport --db=mongo-crud --collection=people --type=csv --headerline --file=data/people.csv
```

If we want to clear the collection before the import, we pass the `--drop` flag.

Run this script by typing:

 ```bash
 sh path_to_file.sh
 ```

Now that we've inserted data into it, the `mongo-crud` database and the `people`
collection both exist.

```bash
$ mongo mongo-crud
MongoDB shell version v3.x.x
connecting to: mongo-crud
> show dbs
local       0.000GB
mongo-crud  0.000GB
> show collections
people
> db.people.count();
2438
```

### Lab: Import Ingredients and Doctors

On your own, use `mongoimport` to bulk load from `data/doctors.csv` and
`data/ingredients.csv`. Save the commands as `.sh` files and run them from the
terminal (not the Mongo shell!).

## Retrieving Documents from a Collection

- [Querying](https://docs.mongodb.org/getting-started/shell/query/)
  - Overview of retrieving data from MongoDB.
- [Queries](https://docs.mongodb.org/manual/reference/mongo-shell/#queries)
  - More detailed overview on retrieving data.
- [Query Operators](https://docs.mongodb.com/manual/reference/operator/query/)
- [`find`](https://docs.mongodb.org/manual/reference/method/db.collection.find/)
  - Detailed documentation on the `find` collection method.
- [`findOne`](https://docs.mongodb.org/manual/reference/method/db.collection.findOne/)
  - Detailed documentation on the `findOne` collection method.
- [Data aggregation](https://docs.mongodb.org/getting-started/shell/aggregation/)
  - Overview of summarizing documents.
- [`aggregate`](https://docs.mongodb.org/manual/reference/method/db.collection.aggregate/)
  - Detailed documentation on the `aggregate` collection method.

MongoDB uses JSON natively (technically
[BSON](https://docs.mongodb.org/manual/reference/glossary/#term-bson)), which
makes it well suited for JavaScript applications.  Conveniently, MongoDB lets us
specify the JSON as a JavaScript object.

### Demo: Read Books

Let's see some of what we can learn about the books in the database.

```bash
> db.books.find({author: "Ernest Hemingway"}).pretty()
{
 "_id" : ObjectId("583ee3f3e6ae0faa5547068e"),
 "title" : "A Farewell to Arms",
 "author" : "Ernest Hemingway",
 "published_on" : "1986-02-15"
}
{
 "_id" : ObjectId("583ee3f3e6ae0faa5547071a"),
 "title" : "The Sun Also Rises",
 "author" : "Ernest Hemingway",
 "published_on" : "2002-10-20"
}
```

```bash
> db.books.find({published_on: /20/}).count()
36
```

```bash
> db.books.find({published_on: /20/}).sort({title: -1}).limit(2).pretty()
{
 "_id" : ObjectId("583ee3f3e6ae0faa5547072f"),
 "title" : "Wide Sargasso Sea",
 "author" : "Jean Rhys",
 "published_on" : "2011-01-14"
}
{
 "_id" : ObjectId("583ee3f3e6ae0faa55470725"),
 "title" : "Trader",
 "author" : "Charles de Lint",
 "published_on" : "2003-06-23"
}
```

**Note:**   When using the REPL, the `.pretty()` method can be quite helpful.

What do we see?

- MongoDB gave each of our documents a unique ID field, called _id.
- MongoDB doesn't care that some documents have fewer or more attributes.

### Code Along: Read People and Doctors

Together, we'll build a query for our people collection. Let's see if we
can find all people born after a date. How about the number of people under
5 feet tall? What about all the doctors who perform surgery?

### Lab: Read Ingredients

Write a query to get all the ingredients with a unit of `tbsp`.

## Deleting Documents

- [Removing Data](https://docs.mongodb.org/getting-started/shell/remove/)
  - Overview of removing documents from a collection.
- [`remove`](https://docs.mongodb.org/manual/reference/method/db.collection.remove/)
  - Detailed documentation of MongoDB's `remove` collection method.
- [`deleteOne`](https://docs.mongodb.com/manual/reference/method/db.collection.deleteOne/)
  - Detailed documentation of MongoDB's `deleteOne` collection method.
- [`deleteMany`](https://docs.mongodb.com/manual/reference/method/db.collection.deleteMany/)
  - Detailed documentation of MongoDB's `deleteMany` collection method.

If we want to clean up, `db.<collection>.drop();` drops the specified collection
and `db.dropDatabase();` drops the current database.

### Demo: Delete Books

We'll remove a few books from the data-store. There are methods for removing
one entry and multiple entries.

```bash
> db.books.deleteOne({author: "John Irving"})
{ "acknowledged" : true, "deletedCount" : 1 }
```

```bash
> db.books.deleteMany({author: "Sinclair Lewis"})
{ "acknowledged" : true, "deletedCount" : 2 }
```

### Code Along: Delete People and Doctors

Let's remove all the people with a specific `born_on` date and doctors with
`Internal medicine` as their specialty.

### Lab: Delete Ingredients

Remove ingredients that have `ml` as their unit of measure.

## Changing the Data in Documents in a Collection

- [Updating Data](https://docs.mongodb.org/getting-started/shell/update/)
  - Overview of changing documents.
- [`update`](https://docs.mongodb.org/manual/reference/method/db.collection.update/)
  - Detailed documentation of MongoDB's `update` collection method.
- [Update Operators](https://docs.mongodb.org/manual/reference/operator/update/)
  - The different modifications we can make during an update.

### Demo: Update Books

MongoDB makes it easy to add an array of items to a document.  We'll update
some books and give them a correct `published_on` value.

```bash
> db.books.update({title: 1984}, {$set: {published_on: "1949-06-08"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```

```bash
> db.books.update({title: "Slaughterhouse-Five"}, {$set: {published_on: "1969-03-31", book_cover: "brown", pages: 247} })
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```

What happens if we run an `update` command without the `$set` option?

### Code along: Update People and Doctors

Now, let's update some people with a hometown. Let's update some doctors'
specialties.

### Lab: Update Ingredients

Update a couple of ingredients' units.

## Adding a Document to a Collection

- [Inserting data](https://docs.mongodb.org/getting-started/shell/insert/)
  - Overview of adding documents to a collection.
- [`insert`](https://docs.mongodb.org/manual/reference/method/db.collection.insert/)
  - Detailed documentation of MongoDB's `insert` collection method.

Next, we'll use the `insert` collection method to add a few more people.  We'll
save our invocations in `scripts/insert/people.js`.  We'll execute that script
using the `mongo` `load` method.  Let's give these people a middle_initial or a
nick_name. Note that the attributes we choose for these people need not match
those from the data we loaded in bulk.

```bash
> load('practice-scripts/insert/people.js');
```

### Code along: Insert Doctors

Together we'll add a few doctors.

### Lab: Insert Ingredients

Add a few ingredients to the `ingredients` collection using `insert`.

## Additional resources

- [BSON Types](https://docs.mongodb.org/manual/reference/bson-types/)
- [Data Model Examples and Patterns](https://docs.mongodb.com/manual/applications/data-models/)
- [Diagrams and Data Explorer for MongoDB](https://www.dbschema.com/diagrams-and-data-explorer-for-mongodb.html)

## [License](LICENSE)

1. All content is licensed under a CC­BY­NC­SA 4.0 license.
1. All software code is licensed under GNU GPLv3. For commercial use or
    alternative licensing, please contact legal@ga.co.
