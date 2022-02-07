# mernsetup
# ![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png)  SOFTWARE ENGINEERING IMMERSIVE

# Build a Blog JSON API

Your task is to build a blog json api using MongoDB, Mongoose, and Express.

Your app should include the following:

- CRUD routes (ability to create, read, update, and delete blog posts)
- a seed file to create seed data in your database

```sh
cd api-exercise-v1
```

Let's create the package.json file!

```sh
npm init -y
```

Add the ability to use `imports` in your package.json file:

```
"type": "module"
```

We now need to install all npm packages that we will be using e.g. [cors](https://www.npmjs.com/package/cors), [express](https://expressjs.com), [mongoose](https://mongoosejs.com), and [morgan](https://www.npmjs.com/package/morgan):

```sh
npm install cors express mongoose morgan
```

We will use [nodemon](https://nodemon.io) as a development dependency, so we should install it as such:

```sh
npm install nodemon -D
```

Remember to configure your package.json to include the start and dev scripts:

package.json
```sh
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
```

Let's setup the folder and file structure for our server application:

```sh
mkdir db models seed
touch db/connection.js models/post.js seed/data.js server.js .gitignore
```

Don't forget to add the node_modules folder to your .gitignore!

.gitignore
```sh
node_modules
.DS_Store
```

Okay, let's get started by setting up the MongoDB connection configuration by using the Mongoose library:

db/connection.js
```js
import mongoose from 'mongoose'

let MONGODB_URI =
  process.env.PROD_MONGODB ||
  'mongodb://127.0.0.1:27017/postsDevDatabase'

// Uncomment to debug Mongoose queries
// mongoose.set('debug', true)

// Will create indexes in MongoDB by default for faster queries
mongoose.set('useCreateIndex', true)

// This is for Model.findByIdAndUpdate method, specifically the so that { new: true} is the default
// Learn more: https://mongoosejs.com/docs/api.html#model_Model.findByIdAndUpdate
mongoose.set('returnOriginal', false)

// Setup connection for MongoDB
// https://mongoosejs.com/docs/connections.html#connections
mongoose
  .connect(MONGODB_URI, { useUnifiedTopology: true, useNewUrlParser: true })
  .catch((error) => console.error('Error connecting to MongoDB: ', error.message))

// Listen to MongoDB events
// Learn more: https://mongoosejs.com/docs/connections.html#connection-events
mongoose.connection.on('disconnected', () => console.log(`Disconnected from MongoDB!`))

// Listen to any errors while connected to MongoDB
// Learn more: https://mongoosejs.com/docs/connections.html#error-handling
mongoose.connection.on('error', (error) => console.error(`MongoDB connection error: ${error}`))

// Export the connection
export default mongoose.connection
```

We now need to define what a blog post looks like in our database. We can do this by defining a schema:

models/post.js
```js
import mongoose from 'mongoose'
const Schema = mongoose.Schema

const Post = new Schema(
  {
    title: { type: String, required: true },
    imgURL: { type: String, required: true },
    content: { type: String, required: true },
    author: { type: String, required: true },
  },
  { timestamps: true }
)

export default mongoose.model('posts', Post)
```

Now that we have a "blueprint" for what a blog post looks like in our database, we can now create some blog posts:

seed/data.js
```js
import db from '../db/connection.js'
import Post from '../models/post.js'

const insertData = async () => {
  // reset database
  await db.dropDatabase()

  const posts =
    [
      {
        "title": "abstract art #1",
        "imgURL": "https://images.unsplash.com/photo-1573521193826-58c7dc2e13e3?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=800&q=60",
        "content": "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.",
        "author": "Jane Doe"
      },
      {
        "title": "abstract art #2",
        "imgURL": "https://images.unsplash.com/photo-1573521193826-58c7dc2e13e3?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=800&q=60",
        "content": "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.",
        "author": "John Doe"
      }
    ]

  await Post.insertMany(posts)
  console.log("Created posts!")

  // close database connection. done.
  db.close()
}

insertData()
```

Run the seed file:

```sh
node seed/data.js
```

Confirm that the MongoDB database (postsMongoDatabase) along with the two blog posts got created in MongoDB by using MongoDB Compass to check.

Next, let's setup our express app and build full CRUD functionality.

server.js
```js
import express from 'express'
import logger from 'morgan'

const PORT = process.env.PORT || 3000

import db from './db/connection.js'
import Post from './models/post.js'

const app = express()

app.use(express.json())
app.use(logger('dev'))

db.on('connected', () => {
  console.log('Connected to MongoDB!')
  app.listen(PORT, () =>
    console.log(`Express server application is running on port ${PORT}`)
  )
})

app.get('/', (req, res) => res.send("This is root!"))

app.get('/posts', async (req, res) => {
    try {
        const posts = await Post.find()
        res.json(posts)
    } catch (error) {
        res.status(500).json({ error: error.message })
    }
})

app.get('/posts/:id', async (req, res) => {
    try {
        const { id } = req.params
        const post = await Post.findById(id)
        if (!post) throw Error('Post not found')
        res.json(post)
    } catch (e) {
        console.log(e)
        res.send('Post not found!')
    }
})

app.post('/posts', async (req, res) => {
    try {
        const post = await new Post(req.body)
        await post.save()
        res.status(201).json(post)
    } catch (error) {
        console.log(error)
        res.status(500).json({ error: error.message })
    }
})

app.put('/posts/:id', async (req, res) => {
  const { id } = req.params
  const post = await Post.findByIdAndUpdate(id, req.body, { new: true })
  res.status(200).json(post)
})

app.delete('/posts/:id', async (req, res) => {
    try {
        const { id } = req.params;
        const deleted = await Post.findByIdAndDelete(id)
        if (deleted) {
            return res.status(200).send("Post deleted")
        }
        throw new Error("Post not found")
    } catch (error) {
        res.status(500).json({ error: error.message })
    }
})
```

Run the server:
```sh
npm run dev
```

**Test all routes in Postman.**

If all CRUD routes are working locally on localhost, then you are ready to deploy.

### Deploy to Heroku using MongoDB Atlas Connection String

> Make sure you're on the `master` or `main` branch!

1. `git status` make sure everything is added and commited.
2. `heroku create your-heroku-app-name`
3. Grab your mongodb uri connection string from MongoDB Cloud Atlas dashboard. 
    - Make sure it works by using MongoDB Compass to test it.
4. `heroku config:set PROD_MONGODB="<INSERT YOUR MONGODB URI CONNECTION STRING HERE>"`
    - change `<password>` to your MongoDB Atlas password and change the database to `postsProductionDatabase` in the connection string
5. `git push heroku master` or `git push heroku main`
    > If there are any errors, debug with `heroku logs --tail`
6. `heroku run node seed/data.js`
    > Now check that your MongoDB Atlas database exists and has data (use MongoDB Compass for this)

Once the build is complete and seeded, test all CRUD routes on Postman: 

- https://your-heroku-app-name.herokuapp.com/posts

## Pull Request when done.
> include the heroku deployment URL in your Pull Request description

## Bonus

Let's create a more intricate schema for a blog post:

```js
import mongoose from 'mongoose'
const Schema = mongoose.Schema

const Post = new Schema(
  {
    title: { type: String, maxLength: 140, required: true },
    photos: [
      {
        name: { type: String, maxLength: 80, required: true },
        imgURL: { type: String, maxLength: 140, required: true },
        location: { type: String, required: true },
      },
    ],
    content: { type: String, maxLength: 2080, required: true },
    author: {
      name: { type: String, maxLength: 80, required: true },
      email: { type: String, maxLength: 140, unique: true, required: true },
      bio: { type: String, maxLength: 240, required: true },
    },
    isPublished: { type: Boolean, default: true },
    category: [
      {
        type: String,
        enum: ['nature', 'coding', 'food'],
        required: true,
      },
    ],
  },
  { timestamps: true }
)

export default mongoose.model('posts', Post)
```

Resource
- https://mongoosejs.com/docs/schematypes.html

Based on the schema above, update your seed file to accommodate the schema.

You will have to delete your database in MongoDB Compass and re-run your seed file.

Once you have your seed file working locally. You can do another deployment to heroku.

You will have to use MongoDB Compass to delete your postsProductionDatabase on Atlas.

Then re-seed on Heroku e.g. `heroku run node seed/data.js`

**Done.**
