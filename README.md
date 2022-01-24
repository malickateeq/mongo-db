
# Mongo DB

## What is it?
    - Schemaless database
    - Database > Collections (like Tables) > Documents (like rows)
    - It uses JSON data format to store in docuements.
    - Mongo DB is flexible and you should be responsible to keep it clean.
    - MongoDB: JSON -> BSON

- Installation
    1. Download `MongoDB Community Server` as .msi under On-premises on mongodb.com website
    2. Install the .msi file
    3. Add mongo bin to windows PATH variable
    4. Create db dir in `C:/Data/db` all DBs will be saved here
    5. Run `mongod` in CMD then `mongo`
    6. Install MongoDB admin toold GUI: Mongo DB Compass

* MongoDB drivers are available for different languages, they are almost the same just the syntax is different. You can learn Shell commands and everything is built on top of it.

## Mongo CLI

- Connect with database
```shell
    mongo   #to switch into mongo environment
    show dbs    #this will list down all databases
    use databaseName    #this will switch to a specific database
    #Now `db` stores current connected database 
```

### CRUD with Mongo
```shell
    # Create
    insertOne(data, options)
    insertMany(data, options)

    # Read
    findOne(filter, options)
    find(filter, options)

    # Update
    updateOne(filter, options)
    replaceOne(filter, options)
    update(filter, options)

    # Delete
    deleteOne(filter, options)
    delete(filter, options)
```

### JS CRUD
```js
    // Create
    db.users.insert({ id: "1122", name: "Test User" })
    db.users.insertMany([{ id: "1122", name: "Test User" }, { id: "1123", name: "Testing User" }])

    // Read
    db.users.find()

    // Update
    db.users.updateOne({ id: "1122" }, { $set: { name: "Malik", type: "admin" } })  // Update name, and type
    
    db.users.updateMany({}, { $set: { status: "active" } }) // Update all records' status to 'active'
    // VS
    db.users.update({}, { status: "active" }) // Update all records' status to 'active' other fields with be replaced with nulled

    // Delete
    db.users.deleteOne({ id: "1122" })
    db.users.deleteMany({ type: "admin" })  // Delete where type is admin
    db.users.deleteMany({})  // Delete all
```

### JS Conditional Fetch
```js
    db.users.find({ age: { $gt: 18 } })     // Find users with age greater than 18
    db.users.find({ age: { $gt: 18 } })     // Find users with age greater than 18
    
    db.users.find({hobbies: "active"})  // Find from hobbies array
    db.users.find({ "status.reason": "active"})  // Find from status nested document
```

### Cursors in MongoDB
```js
    db.users.find() // Will return data with cursor object
    db.users.find().toArray() // Will return all data
```

### Projections (SELECT in SQL)
```js
    db.users.find({}, { name: 1 })  // Will return id and name attributes
```

### Embedded Documents
- Up to 100 Levels of Nesting
- Max 16MB / document
- You can have array of embedded documents
```js
    // Embedd single document
    db.users.updateOne({id: "1122"}, { 
        $set: { 
            status: { type: "suspended", reason: "Invalid IP addresss" } 
        } 
    })

    // Embedd arrays
    db.users.updateOne({id: "1122"}, { 
        $set: { 
            hobbies: [ "Movies", "Football, Gardening" ] 
        }
    })

    // Embedd array of documents
    db.users.updateOne({id: "1122"}, { 
        $set: { 
            hobbies: [ { name: "Gardening" }, { name: "Movies" } ] 
        }
    })
```

## Database Modeling or Schema

### Data Types
- Text, 
- Boolean, 
- Number: Integer (int32), NumberLong (int64), NumberDecimal
- ObjectId: for a unique ID provided by MongoDB
- ISODate("2018-09-09), Timestamp(11421532)
- Embedded documents: { abc: {} }
- Embedd Arrays: { abc: [...] }

### Relationships

#### One to One Relatioship

- Option#1 Embedded approach
```js
    // 1 Database schema

    // "Users" collection
    {
        id: "1",
        name: "Malik",
        car: {
            id: "22",
            model: 2020,
            brand: "Audi"
        }
    }
    // Fetching relationship
    db.users.findOne({}).address;
```

- Option#2 separate relationship
```js
    // 1 Database schema

    // "Users" collection
    {
        id: 1,
        name: "Malik",
    }
    
    // "Cars" collection
    {
        id: "22",
        userId: 1,
        model: 2020,
        brand: "Audi"
    }

    // Fetching relationship
    
```

#### One to Many Relatioship

- Option#1 Embedded approach
```js
    // 1 Database schema

    // "Question" collection
    {
        id: 1,
        creator: "Malik",
        question: "Why snow falls?",
        answers: [
            {
                id: ...,
                text: "Like that!",
            },
            {
                id: ...,
                text: "And like that!",
            }
        ]
    }
```

- Option#2 Separate Collections
```js
    // 1 Database schema

    // "Cities" collection
    {
        id: 1,
        name: "Islamabad",
    }

    // "Citizens" collection
    {
        id: 1,
        cityId: ObjectId(...),
        name: "Malik",
    }
```

#### Many to Many Relatioship

- Option#1 SQLish way
```js
    // 1 Database schema

    // "Customers" collection
    {
        id: 1,
        name: "Test User",
    }

    // "Products" collection
    {
        id: 1,
        name: "MS Office",
        cityId: ObjectId(...),
    }

    // "Orders" collection
    {
        id: 1,
        productId: ObjectId(...),
        customerId: ObjectId(...),
    }
```

- Option#2 Monogo way
```js
    // 1 Database schema

    // "Customers" collection
    {
        id: 1,
        name: "Test User",
        orders: [ { productId: ObjectId(...), status: "pending" } ]
    }

    // "Products" collection
    {
        id: 1,
        name: "MS Office",
        cityId: ObjectId(...),
    }
```

- Option#3 Cloning
- !! Disadvantage of data duplication.
```js
    // 1 Database schema

    // "Customers" collection
    {
        id: 1,
        name: "Test User",
        orders: [ { title: "MS Office", status: "pending" } ]
    }

    // "Products" collection
    {
        id: 1,
        name: "MS Office",
        cityId: ObjectId(...),
    }
```

### Schema Validation

- You can add validation before creating schema and even after creating schema by using `runCommand`
```js
    db.createCollection("posts", { 
        validator: 
        {
            $jsonSchema: {
                bsonType: "object",
                required: ["title", "creator", "comments"],
                properties: {
                    title: { bsonType: "string", description: "Must be a string and is required!" },
                    creator: { bsonType: "objectId", description: "Must be a valid reference ID!" },
                    comments: {
                        bsonType: "array",
                        description: "Must be an array!".
                        items: {
                            bsonType: "object",
                            required: ["text", "author"],
                            properties: {
                                text: { bsonType: "string", description: "Must be a string!" },
                                author: { bsonType: "objectId", description: "Must be a valid reference ID!" },
                            }
                        }
                    }
                },
            }
        },
        validationAction: "warn"
    })

```


### Data Aggregation

#### $lookup
- Lookup allows you to fetch two related documents, merged together in one document in one step.
```js
    db.books.aggregate([{ 
        $lookup: { 
            from: "authors",    // The related collection
            localField: "authors",   // or "authorId" the foreign key in current table
            foreignField: "_id",      // Field in the target collection
            as: "creators"  // The alias you want to name it
        }
    }])

```







- Connect Express with Mongo DB
    1. Mongo DB Driver and Documentations: https://docs.mongodb.com/drivers/node/current/
    2. Install `npm i mongodb` driver to connect with Mongo DB


## Mongo DB Operations

### CREATE
```js


```


## Mongo DB Usage
```js
const mongodb = require("mongodb");
const MongoClient = mongodb.MongoClient;
const connectionURL = "mongodb://127.0.0.1:27017";
const database = "vaultspay-db";
const client = new MongoClient(connectionURL);

// Establish Connection
try 
{
    // Connect with client DB
    client.connect();
    const db = client.db(database);
} 
catch (error) 
{
    return console.log( error );
}
```

- The ObjectID of a collections instance consists of three parts: time stamp, random number and ...

#### Create field
```js
db.collection("users").insertOne({
    name: "Malik",
    age: 40
}, (res, err) => {
    if( err )
    {
        return console.log( err );
    }
    console.log( res );
});
```

#### Read data
```js
db.collection("users").findOne({
    name: "Malik",
    // Or ID etc
}, 
(err, record) => {
    if( err )
    {
        return console.log( err );
    }
    console.log( record );
});

// Fectch collection
db.collection("users").find({name: "Malik"}).toArray((err, users) => {

    if( err )
    {
        return console.log( err );
    }
    console.log( users );
});
```
- For Other DB Operations check MongoDB documentation

### Mongoose Library
- Install: `npm i mongoose`
