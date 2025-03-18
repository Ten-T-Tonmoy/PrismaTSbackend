# PostgreSQL + Prisma: Complete Beginner's Guide

This guide walks you through setting up a Node.js project with PostgreSQL and Prisma ORM, from initial setup to performing CRUD operations. Each step includes explanations suitable for beginners.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Setup Process](#setup-process)
  - [Project Initialization](#project-initialization)
  - [PostgreSQL Setup](#postgresql-setup)
  - [Prisma Integration](#prisma-integration)
- [Database Schema Definition](#database-schema-definition)
- [CRUD Operations](#crud-operations)
  - [Create Records](#create-records)
  - [Read Records](#read-records)
  - [Update Records](#update-records)
  - [Delete Records](#delete-records)
- [Common Issues and Solutions](#common-issues-and-solutions)
- [Next Steps](#next-steps)

## Prerequisites

Before starting, ensure you have:

- [Node.js](https://nodejs.org/) (v14 or newer)
- [PostgreSQL](https://www.postgresql.org/download/) installed and running
- Basic understanding of JavaScript/TypeScript
- Terminal/command line interface

## Setup Process

### Project Initialization

Start by creating a new Node.js project:

```bash
# Create a new directory for your project
mkdir prisma-postgres-demo
cd prisma-postgres-demo

# Initialize a new Node.js project
npm init -y

# Install required dependencies
npm install @prisma/client
npm install prisma --save-dev
```

**Why these commands?**

- `mkdir` creates a new directory for your project
- `npm init -y` generates a default package.json file without prompting for inputs
- We install Prisma client as a regular dependency (for runtime) and Prisma CLI as a dev dependency (for development)

### PostgreSQL Setup

1. Ensure PostgreSQL is running on your system
2. Create a new database:

```bash
# Connect to PostgreSQL
psql -U postgres

# Create a new database (in the PostgreSQL shell)
CREATE DATABASE prisma_demo;

# Exit the PostgreSQL shell
\q
```

**Why this step?**

- We need a database for our application to connect to
- Using the PostgreSQL CLI (`psql`) lets us interact directly with the database server
- Creating a dedicated database isolates our application data

### Prisma Integration

Initialize Prisma in your project:

```bash
# Initialize Prisma in your project
npx prisma init
```

This creates:

- A `prisma` directory with a `schema.prisma` file
- A `.env` file for environment variables

Edit the `.env` file to add your PostgreSQL connection string:

```
DATABASE_URL="postgresql://postgres:password@localhost:5432/prisma_demo?schema=public"
```

**Format explanation:**

- `postgresql://` - Database provider
- `postgres:password` - Username:password
- `@localhost:5432` - Host and port
- `/prisma_demo` - Database name
- `?schema=public` - Schema name

**Why this step?**

- The `prisma init` command sets up the necessary files for Prisma
- The connection string tells Prisma how to connect to your PostgreSQL database
- Environment variables keep sensitive info out of your code

## Database Schema Definition

Edit the `prisma/schema.prisma` file to define your data model:

```prisma
// This is your Prisma schema file

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Define your models here
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  Int
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

**Why this schema?**

- `generator client` specifies that we want to generate a JavaScript client
- `datasource db` configures the database connection
- The `User` and `Post` models create tables in our database
- We define a relationship between users and posts
- Special fields like `@id` and `@unique` define constraints
- `@default`, `@now()` and `@updatedAt` provide automatic values

Now create the database tables with this command:

```bash
# Create the tables in the database
npx prisma migrate dev --name init
```

**Why this command?**

- `prisma migrate dev` creates and applies a migration to your database
- `--name init` names the migration "init" for tracking purposes
- This command also generates the Prisma Client tailored to your schema

## CRUD Operations

Create a new file called `index.js` to work with your database:

```javascript
// index.js
const { PrismaClient } = require("@prisma/client");
const prisma = new PrismaClient();

async function main() {
  // Your CRUD operations will go here

  // Create example
  const user = await createUser("alice@example.com", "Alice");
  console.log("Created user:", user);

  const post = await createPost("My First Post", "Hello, world!", user.id);
  console.log("Created post:", post);

  // Read example
  const users = await getUsers();
  console.log("All users:", users);

  const userWithPosts = await getUserWithPosts(user.id);
  console.log("User with posts:", userWithPosts);

  // Update example
  const updatedUser = await updateUser(user.id, { name: "Alice Smith" });
  console.log("Updated user:", updatedUser);

  // Delete example
  const deletedPost = await deletePost(post.id);
  console.log("Deleted post:", deletedPost);
}

// CRUD functions

// Create operations
async function createUser(email, name) {
  return prisma.user.create({
    data: {
      email,
      name,
    },
  });
}

async function createPost(title, content, authorId) {
  return prisma.post.create({
    data: {
      title,
      content,
      authorId,
      published: true,
    },
  });
}

// Read operations
async function getUsers() {
  return prisma.user.findMany();
}

async function getUserWithPosts(id) {
  return prisma.user.findUnique({
    where: { id },
    include: { posts: true },
  });
}

// Update operations
async function updateUser(id, data) {
  return prisma.user.update({
    where: { id },
    data,
  });
}

// Delete operations
async function deletePost(id) {
  return prisma.post.delete({
    where: { id },
  });
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    // Close the database connection when done
    await prisma.$disconnect();
  });
```

Run your script:

```bash
node index.js
```

**Why this structure?**

- We import the `PrismaClient` to interact with the database
- The `main()` function uses async/await to handle asynchronous database operations
- Each CRUD operation is isolated in its own function for clarity
- We properly handle errors and close the database connection

### Create Records

```javascript
// Create a new user
async function createUser(email, name) {
  return prisma.user.create({
    data: {
      email,
      name,
    },
  });
}

// Create a post for a user
async function createPost(title, content, authorId) {
  return prisma.post.create({
    data: {
      title,
      content,
      authorId,
      published: true,
    },
  });
}
```

**Why it works:**

- `prisma.user.create()` inserts a new record into the User table
- The `data` object contains the values for the new record
- Prisma handles SQL generation and parameter binding securely

### Read Records

```javascript
// Get all users
async function getUsers() {
  return prisma.user.findMany();
}

// Get a specific user with their posts
async function getUserWithPosts(id) {
  return prisma.user.findUnique({
    where: { id },
    include: { posts: true },
  });
}
```

**Why it works:**

- `findMany()` retrieves multiple records (similar to SELECT \* FROM users)
- `findUnique()` gets a single record by its unique identifier
- `include` enables eager loading of relationships (getting posts with the user)

### Update Records

```javascript
// Update a user's information
async function updateUser(id, data) {
  return prisma.user.update({
    where: { id },
    data,
  });
}
```

**Why it works:**

- `update()` modifies an existing record
- `where` specifies which record to update
- `data` contains the fields to update and their new values

### Delete Records

```javascript
// Delete a post
async function deletePost(id) {
  return prisma.post.delete({
    where: { id },
  });
}
```

**Why it works:**

- `delete()` removes a record from the database
- `where` specifies which record to delete
- Prisma handles SQL generation for deletion

## Common Issues and Solutions

1. **Connection errors**:

   - Check if PostgreSQL is running
   - Verify your connection string in `.env`
   - Make sure the user has necessary permissions

2. **Migration errors**:

   - Run `npx prisma migrate reset` to reset and reapply migrations
   - Check your schema for syntax errors

3. **Relation errors**:
   - Ensure foreign keys reference existing records
   - Check your relation field definitions

## Next Steps

Now that you have a basic understanding of Prisma with PostgreSQL, you can:

1. Expand your schema with more models and relationships
2. Implement API endpoints using Express or another framework
3. Add validation and error handling
4. Learn about Prisma's advanced features:
   - Transactions
   - Raw database access
   - Middleware
   - Full-text search

For more information, check the [Prisma documentation](https://www.prisma.io/docs/).

---

Happy coding! 🚀
