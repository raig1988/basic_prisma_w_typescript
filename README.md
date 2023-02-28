# basic_prisma_w_typescript
Basic creation of prisma api with typescript as described in getting started

https://www.prisma.io/docs/getting-started/quickstart

Get started
Quickstart
In this Quickstart guide, you'll learn how to get started with Prisma from scratch using a plain TypeScript project and a local SQLite database file. It covers data modeling, migrations and querying a database.

If you want to use Prisma with your own PostgreSQL, MySQL, MongoDB or any other supported database, go here instead:

Start with Prisma from scratch
Add Prisma to an existing project
Prerequisites
You need Node.js v14.17.0 or higher for this guide (learn more about system requirements).

1. Create TypeScript project and set up Prisma
As a first step, create a project directory and navigate into it:

mkdir hello-prisma 
cd hello-prisma 
Next, initialize a TypeScript project using npm:

npm init -y 
npm install typescript ts-node @types/node --save-dev 
This creates a package.json with an initial setup for your TypeScript app.

Now, initialize TypeScript:

npx tsc --init 
Then, install the Prisma CLI as a development dependency in the project:

npm install prisma --save-dev 
Finally, set up Prisma with the init command of the Prisma CLI:

npx prisma init --datasource-provider sqlite 
This creates a new prisma directory with your Prisma schema file and configures SQLite as your database. You're now ready to model your data and create your database with some tables.

2. Model your data in the Prisma schema
The Prisma schema provides an intuitive way to model data. Add the following models to your schema.prisma file:

prisma/schema.prisma
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}

model Post {
  id        Int     @id @default(autoincrement())
  title     String
  content   String?
  published Boolean @default(false)
  author    User    @relation(fields: [authorId], references: [id])
  authorId  Int
}
Models in the Prisma schema have two main purposes:

Represent the tables in the underlying database
Serve as foundation for the generated Prisma Client API
In the next section, you will map these models to database tables using Prisma Migrate.

3. Run a migration to create your database tables with Prisma Migrate
At this point, you have a Prisma schema but no database yet. Run the following command in your terminal to create the SQLite database and the User and Post tables represented by your models:

npx prisma migrate dev --name init 
This command did two things:

It creates a new SQL migration file for this migration in the prisma/migrations directory.
It runs the SQL migration file against the database.
Because the SQLite database file didn't exist before, the command also created it inside the prisma directory with the name dev.db as defined via the environment variable in the .env file.

Congratulations, you now have your database and tables ready. Let's go and learn how you can send some queries to read and write data!

4. Explore how to send queries to your database with Prisma Client
To send queries to the database, you will need a TypeScript file to execute your Prisma Client queries. Create a new file called script.ts for this purpose:

touch script.ts 
Then, paste the following boilerplate into it:

script.ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  // ... you will write your Prisma Client queries here
}

main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })
This code contains a main function that's invoked at the end of the script. It also instantiates PrismaClient which represents the query interface to your database.

4.1. Create a new User record
Let's start with a small query to create a new User record in the database and log the resulting object to the console. Add the following code to your script.ts file:

script.ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  const user = await prisma.user.create({
    data: {
      name: 'Alice',
      email: 'alice@prisma.io',
    },
  })
  console.log(user)
}

main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })
Instead of copying the code, you can type it out in your editor to experience the autocompletion Prisma Client provides. You can also actively invoke the autocompletion by pressing the CTRL+SPACE keys on your keyboard.

Next, execute the script with the following command:

npx ts-node script.ts 
Hide CLI results
{ id: 1, email: 'alice@prisma.io', name: 'Alice' }
Great job, you just created your first database record with Prisma Client! 🎉

In the next section, you'll learn how to read data from the database.

4.2. Retrieve all User records
Prisma Client offers various queries to read data from your database. In this section, you'll use the findMany query that returns all the records in the database for a given model.

Delete the previous Prisma Client query and add the new findMany query instead:

script.ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  const users = await prisma.user.findMany()
  console.log(users)
}

main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })
Execute the script again:

npx ts-node script.ts 
Hide CLI results
[{ id: 1, email: 'alice@prisma.io', name: 'Alice' }]
Notice how the single User object is now enclosed with square brackets in the console. That's because the findMany returned an array with a single object inside.

4.3. Explore relation queries with Prisma
One of the main features of Prisma Client is the ease of working with relations. In this section, you'll learn how to create a User and a Post record in a nested write query. Afterwards, you'll see how you can retrieve the relation from the database using the include option.

First, adjust your script to include the nested query:

script.ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  const user = await prisma.user.create({
    data: {
      name: 'Bob',
      email: 'bob@prisma.io',
      posts: {
        create: {
          title: 'Hello World',
        },
      },
    },
  })
  console.log(user)
}

main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })
Run the query by executing the script again:

npx ts-node script.ts 
Hide CLI results
{ id: 2, email: 'bob@prisma.io', name: 'Bob' }
By default, Prisma only returns scalar fields in the result objects of a query. That's why, even though you also created a new Post record for the new User record, the console only printed an object with three scalar fields: id, email and name.

In order to also retrieve the Post records that belong to a User, you can use the include option via the posts relation field:

script.ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  const usersWithPosts = await prisma.user.findMany({
    include: {
      posts: true,
    },
  })
  console.dir(usersWithPosts, { depth: null })
}

main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })
Run the script again to see the results of the nested read query:

npx ts-node script.ts 
Hide CLI results
[
  { id: 1, email: 'alice@prisma.io', name: 'Alice', posts: [] },
  {
    id: 2,
    email: 'bob@prisma.io',
    name: 'Bob',
    posts: [
      {
        id: 1,
        title: 'Hello World',
        content: null,
        published: false,
        authorId: 2
      }
    ]
  }
]
This time, you're seeing two User objects being printed. Both of them have a posts field (which is empty for "Alice" and populated with a single Post object for "Bob") that represents the Post records associated with them.

Notice that the objects in the usersWithPosts array are fully typed as well. This means you will get autocompletion and the TypeScript compiler will prevent you from accidentally typing them.

5. Next steps
In this Quickstart guide, you have learned how to get started with Prisma in a plain TypeScript project. Feel free to explore the Prisma Client API a bit more on your own, e.g. by including filtering, sorting, and pagination options in the findMany query or exploring more operations like update and delete queries.

Explore the data in Prisma Studio
Prisma comes with a built-in GUI to view and edit the data in your database. You can open it using the following command:

npx prisma studio 
Set up Prisma with your own database
If you want to move forward with Prisma using your own PostgreSQL, MySQL, MongoDB or any other supported database, follow the Set Up Prisma guides:

Start with Prisma from scratch
Add Prisma to an existing project
Explore ready-to-run Prisma examples
Check out the prisma-examples repository on GitHub to see how Prisma can be used with your favorite library. The repo contains examples with Express, NestJS, GraphQL as well as fullstack examples with Next.js and Vue.js, and a lot more.

Build an app with Prisma
The Prisma blog features comprehensive tutorials about Prisma, check out our latest ones:

Build a fullstack app with Remix (5 parts, including videos)
Build a REST API with NestJS
Join the Prisma community 💚
Prisma has a huge community of developers. Join us on Slack or ask questions via GitHub Discussions.
