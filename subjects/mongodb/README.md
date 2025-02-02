# MongoDB

Learn the basics of [MongoDB][mongodb], one of the most populars document-oriented databases.

**You will need**

* A Unix CLI
* A running MongoDB server ([installation instructions][install])

**Recommended reading**

* [Command line](../cli/)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [What is MongoDB?](#what-is-mongodb)
  - [Key features](#key-features)
  - [Document-oriented database](#document-oriented-database)
  - [Schema-less collections](#schema-less-collections)
- [Query language](#query-language)
  - [Connecting](#connecting)
  - [Inserting documents](#inserting-documents)
  - [Inserting multiple documents](#inserting-multiple-documents)
  - [Finding documents](#finding-documents)
    - [Query operators](#query-operators)
    - [Projection](#projection)
    - [Sort, skip & limit](#sort-skip--limit)
  - [Counting documents](#counting-documents)
  - [Updating documents](#updating-documents)
    - [Updating multiple documents](#updating-multiple-documents)
    - [Upserts](#upserts)
    - [Upsert behavior](#upsert-behavior)
    - [Atomic operations](#atomic-operations)
  - [Replacing documents](#replacing-documents)
  - [Removing documents](#removing-documents)
- [Indexes](#indexes)
  - [Unique indexes](#unique-indexes)
  - [Query performance](#query-performance)
  - [Improving performance with indexes](#improving-performance-with-indexes)
  - [Index scan](#index-scan)
  - [Performance of compound indexes](#performance-of-compound-indexes)
- [Resources](#resources)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->



## What is MongoDB?

<!-- slide-front-matter class: center, middle, image-header -->

<p class='center'><img src='images/mongodb.png' width='60%' /></p>

> "MongoDB is a free and open-source cross-platform **document-oriented database** program."



### Key features

* High performance
  * Atomic operations
  * Unacknowledged writes
* **Rich query language**
  * **Data aggregation**
  * Full text search
  * Geospatial queries
* High availability
  * Replica sets with automatic failover and data redundancy
* Horizontal scalability
  * Sharding
  * Zones



### Document-oriented database

Instead of SQL rows, you store [**JSON-like**][bson] documents in MongoDB:

```json
{
  "_id" : ObjectId("54c955492b7c8eb21818bd09"),
  "firstName": "John",
  "lastName": "Doe",
  "birthDate": ISODate("1970-10-01T00:00:00Z"),
  "interests": [ "Pastry", "Kung fu" ],
  "address" : {
    "city": "Livingston",
    "street" : "13 Garden Street",
    "zip" : "07039"
  },
  "phones": [
    { "type": "professional", "number": "+1-202-555-0144" },
    { "type": "home", "number": "+1-202-555-0186" }
  ]
}
```

* Documents are **key-value** data structures
* Values may include other **documents, arrays, and arrays of documents**



### Schema-less collections

Documents are stored in **collections**.
Unlike SQL tables, collections are **schema-less**.
These two documents could be stored in the same collection:

```json
{
  "_id" : ObjectId("54c955492b7c8eb21818bd09"),
  "name": "John Doe",
  "birthDate": ISODate("1970-10-01T00:00:00Z"),
  "interests": [ "Pastry", "Kung fu" ]
}
```

```json
{
  "_id" : ObjectId("54c955492b7c8eb21818cd1"),
  "model": "Campagna T-Rex",
  "wheels": 3,
  "dimensions": { "length": "3.5m", "width": "1.981m", "height": "1.067m" }
}
```

This example is probably not a good idea, but it also means your documents can **evolve** (add new keys, remove others) without having to migrate your schema.

In MongoDB, documents stored in a collection must have a unique `_id` field that acts as a **primary key**.



## Query language

<!-- slide-front-matter class: center, middle -->

Inserting, querying, updating and removing documents.



### Connecting

Connect to the MongoDB shell from your CLI and you should have a **new prompt**, indicating that you can now type MongoDB commands and queries:

```bash
$> mongo
Connecting to:		mongodb://127.0.0.1:27017/
Using MongoDB:		6.0.1
Using Mongosh:		1.5.4
...
test>
```

You can switch databases with `use <name>`. Let's do that now:

```bash
test> use test
already on db test
test> use archioweb
switched to db archioweb
```

Note that you do not need to create the "archioweb" database before accessing it.
It is **automatically created** as you first access it.



### Inserting documents

Insert a couple of documents into a collection named **people** (again, it is automatically created if it doesn't exist):

```bash
db.people.insertOne({
  "name": "John Doe",
  "birthDate": ISODate("1970-10-01T00:00:00Z"),
  "children": 2,
  "address" : { "city": "Livingston", "street" : "13 Garden Street" },
  "interests": [ "Pastry", "Kung fu" ],
  "phones": []
})
db.people.insertOne({
  "name": "John Smith",
  "birthDate": ISODate("1990-12-24T00:00:00Z"),
  "address" : { "city": "Newport", "street" : "85 Bay Drive" },
  "interests": [ "Sunglasses" ],
  "phones": [
    { "type": "professional", "number": "+1-202-555-0144" },
    { "type": "home", "number": "+1-202-555-0186" }
  ]
})
```

MongoDB should acknowledge each insertion by showing you the new `ObjectId` of
the inserted object.

### Inserting multiple documents

You can insert multiple documents at once by using the
[`insertMany`][mongodb-insert-many] method and passing it an array.

```bash
db.people.insertMany([{
  "name": "Saul Goodman",
  "birthDate": ISODate("1970-10-01T00:00:00Z"),
  "children": 2,
  "address" : { "city": "Albuquerque", "street" : "13 Garden Street" },
  "interests": [ "Law", "Money" ],
  "phones": []
  },
  {
  "name": "Jimmy McGill",
  "birthDate": ISODate("1970-10-01T00:00:00Z"),
  "children": 2,
  "address" : { "city": "Albuquerque", "street" : "13 Garden Street" },
  "interests": [ "Family", "Falling" ],"phones": []
  }
])
```

### Finding documents

Here are a few simple queries you can run with MongoDB's [find][find] method:

```js
// Find all people
db.people.find({})

// Find all people named John Doe
db.people.find({ "name": "John Doe" })

// Find all people living in Newport
db.people.find({ "address.city": "Newport" })

// Find all people that have a home phone number
db.people.find({ "phones.type": "home" })

// Find all people interested in Kung fu
db.people.find({ "interests": "Kung fu"  })

// Find all people named John Smith AND living in Livingston
db.people.find({ "name": "John Smith", "address.city": "Livingston" })
```

#### Query operators

You can write more complex queries with query operators:

```js
// Find all people living in Newport or Livingston
db.people.find({ "address.city": { "$in": [ "Newport", "Livingston" ] } })

// Find all people born after 1980
db.people.find({ "birthDate": { "$gt": ISODate("1980-01-01")  }  })

// Find all people with no phone numbers
db.people.find({ "phones": { "$size": 0  }  })

// Find all people named John Smith OR living in Livingston
db.people.find({
  $or: [
    { "name": "John Smith" },
    { "address.city": "Livingston" }
  ]
})
```

Read the documention on [query operators][query-operators] to learn more about these and other operators.

#### Projection

You can give a second parameter to `find` to specify which fields to return:


```js
// Find all people and get their name and address only
// ("SELECT name, address FROM people" in SQL)
db.people.find({}, { "name": 1, "address": 1 })

// Find all people named John Doe but without their interests or address
db.people.find({ "name": "John Doe" }, { "address": 0, "interests": 0 })
```

You can specify fields to include or fields to exclude, **not both**.

#### Sort, skip & limit

The `sort` method orders the documents in the result set:

```js
// Find all people sorted by descending name
// ("SELECT * FROM people ORDER BY name DESC" in SQL)
db.people.find({}).sort({ "name": -1 })
```

The `skip` and `limit` methods allow you to select a slice of the result set:

```js
// Find one person
// ("SELECT * FROM people LIMIT 1" in SQL)
db.people.find({}).limit(1)

// Find one person among people sorted by name, starting at the second person
// ("SELECT * FROM people ORDER BY name OFFSET 1 LIMIT 1" in SQL)
db.people.find({}).sort({ "name": 1 }).skip(1).limit(1)
```



### Counting documents

Counting documents works basically the same way as finding them:

```js
// Count all people
db.people.countDocuments({})

// Count all people named John Doe
db.people.countDocuments({ "name": "John Doe" })

// Count all people that have a home phone number
db.people.countDocuments({ "phones.type": "home" })

// Count all people named John Smith AND living in Livingston
db.people.countDocuments({ "name": "John Smith", "address.city": "Livingston" })

// Count all people born after 1980
db.people.countDocuments({ "birthDate": { "$gt": ISODate("1980-01-01")  }  })

// Count all people named John Smith OR living in Livingston
db.people.countDocuments({
  $or: [
    { "name": "John Smith" },
    { "address.city": "Livingston" }
  ]
})
```



### Updating documents

Here's an update example:

```js
// Rename "John Smith" to "John A. Smith" and add "Movies" to his interests
db.people.updateOne(
  { "name": "John Smith" },
  {
    "$set": { "name": "John A. Smith" },
    "$push": { "interests": "Movies" }
  }
)
```

The [`updateOne`][mongodb-update-one] method takes 3 parameters:

* A filter to match the documents to update
* An update document to specify the modification to perform
* An options parameter (optional)

Read the documention on [update operators][update-operators] to learn more about these and other operators.

#### Updating multiple documents

To update multiple documents, you must use the [`updateMany`][mongodb-update-many] method.
Otherwise, only the first matching document will be updated.

#### Upserts

The `upsert` option allows you to **automatically insert** a new document when you try to make an **update** but **no matching document is found**:

```js
// Update Ned Stark, but create him if he doesn't exist
db.people.updateOne(
  { "name": "Ned Stark" },
  {
    "$set": {
      "children": 6,
      "interests": [ "Snow" ]
    }
  },
  {
    "upsert": true
  }
)
```

#### Upsert behavior

MongoDB will tell you that an upsert has taken place:

```txt
{
  acknowledged: true,
  insertedId: ObjectId("6311d94c2af6bb4dd5b40a9a"),
  matchedCount: 0,
  modifiedCount: 0,
  upsertedCount: 1
}
```

Note that the query was merged with the update (the new person also has the `name` field):

```js
db.people.find({ "name": "Ned Stark" })

{
  "_id" : ObjectId("589f24f1f0bb4e1d9fedf1eb"),
  "name" : "Ned Stark",
  "children" : 6,
  "interests" : [ "Snow" ]
}
```

#### Atomic operations

Some operators are **atomic operations**:

```js
// Add one child to John Doe
db.people.updateOne(
  { "name": "John Doe" },
  {
    "$inc": {
      "children": 1
    }
  }
)
```

This means that:

* You can perform some arithmetic operations **directly in the database** without having to first fetch data, then update it
* These operations are not subject to **concurrency issues** (at least on a single node)

### Replacing documents

The [`replaceOne`][mongodb-replace-one] and [`replaceMany`][mongodb-replace-many] methods allow you to replace the entirety of a document.

```js
> db.people.replaceOne(
  {name: "Saul Goodman"},
  {
    name: "Gene",
    children: 0,
    address: "N/A"
  }
)

> db.people.find({name: "Gene"})
[
  {
    _id: ObjectId("6311d6ccd2b0f4cf05594185"),
    name: 'Gene',
    children: 0,
    address: 'N/A'
  }
]
```

### Removing documents

You can use [`deleteMany`][mongodb-delete-many] to remove documents from a collection:

```js
// Remove all people
db.people.deleteMany({})

// Remove all people who have 5 children
db.people.deleteMany({ "children": 5 })
```

It removes all matching documents by default.
Use the [`deleteOne`][mongodb-delete-one] mathod if you want to only remove the first matching document:

```js
// Remove the first person found who likes chocolate
db.people.deleteOne({ "interests": "Chocolate" })
```



## Indexes

<!-- slide-front-matter class: center, middle -->

Indexes can support the efficient execution of queries and enforce constraints.



### Unique indexes

A unique index enforces uniqueness for the indexed fields:

```js
// Ensure that there are no two people born on the same day
db.people.createIndex({ "birthDate": 1 }, { "unique": true })

// Ensure that there are no two people with the same name in the same city
db.people.createIndex({ "name": 1, "address.city": 1 }, { "unique": true })
```

By default, MongoDB creates a unique index on the `_id` field during the creation of a collection.

Read the [documentation on indexes][indexes] to find out more about other types of indexes.



### Query performance

Without indexes, MongoDB must perform a **full collection scan** to find the matching documents.
Use `explain` to find out how a query will be executed.
The `COLLSCAN` value in the `winningPlan` object tells us that a collection scan will be performed when executing that query:

```bash
db.people.find({ name: "John A. Smith" }).explain()

{
  "queryPlanner" : {
    "plannerVersion" : 1,
    "namespace" : "test.people",
    "indexFilterSet" : false,
    "parsedQuery" : {
      "name" : { "$eq" : "John A. Smith" }
    },
*   "winningPlan" : {
*     "stage" : "COLLSCAN",
      "filter" : {
        "name" : { "$eq" : "John A. Smith" }
      },
      "direction" : "forward"
    },
    "rejectedPlans" : [ ]
  },
  "ok" : 1
}
```

### Improving performance with indexes

If an appropriate index exists for a query, MongoDB can use the index to **limit the number of documents it must inspect**.

Use [createIndex][create-index] to add an index:

```js
// Add a simple index for queries on the "name" field
db.people.createIndex( { "name": 1 } )

{
  "createdCollectionAutomatically" : false,
  "numIndexesBefore" : 1,
  "numIndexesAfter" : 2,
  "ok" : 1
}
```

### Index scan

Run `explain` again to confirm that your index will be applied (you should see `IXSCAN` in the `winningPlan` object):

```bash
db.people.find({ name: "John A. Smith" }).explain()

{
  "queryPlanner" : {
    "plannerVersion" : 1,
    "namespace" : "demo.people",
    "indexFilterSet" : false,
    "parsedQuery" : {
      "name" : { "$eq" : "John A. Smith" }
    },
*   "winningPlan" : {
      "stage" : "FETCH",
      "inputStage" : {
*       "stage" : "IXSCAN",
        "keyPattern" : { "name" : 1 },
        "indexName" : "name_1",
        ...
      }
    },
    "rejectedPlans" : [ ]
  },
  "ok" : 1
}
```



### Performance of compound indexes

You can create an index on **multiple fields**:

```js
// Create an index of people sorted first from oldest to youngest,
// then by ascending name
db.people.createIndex({ "birthDate": -1, "name": 1 });
```

**Be careful:** for compound indexes, the **order** of fields is important:
it determines what queries the index can support:

```js
// Queries sorted like the index will use it
db.people.find({}).sort({ "birthDate": -1, "name": 1 })

// Reverse order is supported as well
db.people.find({}).sort({ "birthDate": 1, "name": -1 })

// But this is an incompatible sort order
// The index will NOT be used, and MongoDB will revert to a collection scan
db.people.find({}).sort({ "birthDate": 1, "name": 1 })
```







## Resources

* [Getting started with MongoDB][getting-started]
* [db.collection.find][find] & [query operators][query-operators]
* [db.collection.insertOne][mongodb-insert-one] & [db.collection.insertMany][mongodb-insert-many]
* [db.collection.updateOne][mongodb-update-one] & [db.collection.updateOne][mongodb-update-many]
* [update operators][update-operators]
* [db.collection.replaceOne][mongodb-replace-one] & [db.collection.replaceMany][mongodb-replace-many]
* [db.collection.deleteMany][mongodb-delete-many] & [db.collection.deleteOne][mongodb-delete-one]
* [db.collection.createIndex][create-index]
* [Indexes][indexes]
* [SQL comparison][sql-comparison]



[bson]: https://www.mongodb.com/json-and-bson
[create-index]: https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/
[find]: https://docs.mongodb.com/manual/reference/method/db.collection.find/
[getting-started]: https://docs.mongodb.com/getting-started/shell/
[indexes]: https://docs.mongodb.com/manual/indexes/
[install]: https://github.com/MediaComem/comem-archioweb/guides/install-mongodb.md
[query-operators]: https://docs.mongodb.com/manual/reference/operator/query/
[mongodb]: https://www.mongodb.com
[mongodb-insert-one]: https://www.mongodb.com/docs/manual/reference/method/db.collection.insertOne/
[mongodb-insert-many]: https://www.mongodb.com/docs/manual/reference/method/db.collection.insertMany/
[mongodb-delete-one]: https://docs.mongodb.com/manual/reference/method/db.collection.deleteOne/
[mongodb-delete-many]: https://docs.mongodb.com/manual/reference/method/db.collection.deleteMany/
[mongodb-replace-one]: https://www.mongodb.com/docs/manual/reference/method/db.collection.replaceOne/
[mongodb-replace-many]: https://www.mongodb.com/docs/manual/reference/method/db.collection.replaceMany/
[mongodb-update-one]: https://www.mongodb.com/docs/manual/reference/method/db.collection.updateOne/
[mongodb-update-many]: https://www.mongodb.com/docs/manual/reference/method/db.collection.updateMany/
[sql-comparison]: https://docs.mongodb.com/manual/reference/sql-comparison/
[update-operators]: https://docs.mongodb.com/manual/reference/operator/update/
