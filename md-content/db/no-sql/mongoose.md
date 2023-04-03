# Dealing with mongoose

## Setting up connection

To set up connection to db, protected by authentication, one should do next steps:
```
const connection = await mongoose.createConnection('mongodb://host:port/dbName', {
    authSource: 'admin',
    user: 'username',
    pass: 'password',
})
```
or
```
connection = await mongoose.createConnection(`mongodb://username:password@host:port/dbName?authSource=admin`)
```

If we want to use an appropriate client connected to db and be sure that connection is set before client usage,
then on app initialization we can use something like this:
// client.js
```
//...
const databaseClient = await mongoose.createConnection(`mongodb://username:password@host:port/dbName?authSource=admin`)

async function initClient() {
    // if connection failed - exit with error code
    if ((await databaseClient.asPromise()).readyState !== 1) process.exit(1)
    return true
}

module.exports = {
    databaseClient,
    initClient
}
```
// app.js
```
const { databaseClient, initClient } = require('./client')

async function runApp() {
    await initClient()
    // ...
}
```
After this `databaseClient` will be valid and connected to db anywhere it's used.

## Creating relations between documents

Let's say there's a collection `User` and a collection `Task`. User can have related tasks and task should be related to user via user's ID.  
So `Task` document should have a key (in schema, and in db) equal to ID of user having this task. But it would be nice, that user could have a key `tasks` that could return a list of related tasks.  
`mongoose` allows creating relations between collections.

`populate` function is used to get data from related document via appropriate name.

https://mongoosejs.com/docs/api/schema.html#schema_Schema-virtual
https://mongoosejs.com/docs/api/schematype.html#schematype_SchemaType-ref

```
const TaskSchema = new Schema({
    //... props related to task
    owner: Schema.Types.ObjectId // in this example it will be '_id' of User
    required: true,
    ref: 'User' // or can be User - model instance
})

const UserSchema = new Schema({
    //... props related to user
}, {
    // so virtuals could be included to res.json and console.log
    toJSON: { virtuals: true },
    toObject: { virtuals: true }
})

UserSchema.virtual('tasks', {
    ref: 'Task', // or can be Task - model instance
    localField: '_id', // field of User that has a relation in Task
    foreignField: 'owner' // field in Task that relates to local field
})


// ... code using instances of task and user
await task.populate('owner') // now task has object of owner
await user.populate('tasks') // now user has field tasks with tasks objects

```

## Hiding private data
When model instance is used in `res`, on sending it's converted to string using `JSON.strigify`,
so to change how it's converted - modify method that is called in `JSON.strigify`:
```
UserSchema.methods.toJSON = function() {
    const user = this.toObject()

    delete user.privateField

    return user
}
```

## Filtering data in a related document
Let's say there a `User` and `Task` models. `User` has a virtual field `tasks` related to `Task`.  
So to get filtered tasks for a given user, one should do something like this:
```
user.populate({
    path: 'tasks',
    match: {
        completed: true // if one wants to get only completed tasks. 
    }
})
```
In example above we used task's field `completed` for filtering.

## Paginating and sorting data
### In a given document
Let's say there a `Task` model. To get paginated data, use something like this:
```
await Task
    .find()
    .sort({ timestamp: 'asc' })
    .limit(10)
    .skip(20)
    .exec()

// or
await Task.find({}, null, { skip: 10 }).exex()

```
https://mongoosejs.com/docs/api/query.html#query_Query-sort
https://mongoosejs.com/docs/api/query.html#query_Query-limit

### In a related document
Let's say there a `User` and `Task` models. `User` has a virtual field `tasks` related to `Task`.  
So to get paginated tasks for a given user, one should do something like this:
```
await user.populate({
    path: 'tasks',
    options: {
        limit: 10;
        skip: 20
    }
})

await user.populate({
    path: 'tasks'
}).sort({ age: 1 })
```
In example above we used task's field `completed` for filtering.
