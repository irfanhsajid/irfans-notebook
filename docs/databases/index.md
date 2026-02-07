# Databases

A comprehensive guide to working with databases, focusing on MongoDB, PostgreSQL, and MySQL.

## Database Types

### SQL Databases (Relational)

- **PostgreSQL**: Advanced open-source relational database
- **MySQL**: Popular open-source relational database

### NoSQL Databases

- **MongoDB**: Document-oriented database

## Choosing the Right Database

### Use PostgreSQL When:

- You need complex queries and joins
- ACID compliance is critical
- You need advanced features (JSON support, full-text search)
- Data integrity is paramount

### Use MySQL When:

- You need a simple, fast relational database
- You're building a web application
- You want wide hosting support
- Read-heavy operations are common

### Use MongoDB When:

- You have flexible, evolving schemas
- You're working with JSON-like documents
- Horizontal scaling is important
- You need high write throughput

## Common Database Operations

### Connection Examples

#### PostgreSQL (Node.js with pg)

```javascript
const { Pool } = require("pg");

const pool = new Pool({
  user: "username",
  host: "localhost",
  database: "mydb",
  password: "password",
  port: 5432,
});

// Query
const result = await pool.query("SELECT * FROM users WHERE id = $1", [1]);
```

#### MySQL (Node.js with mysql2)

```javascript
const mysql = require("mysql2/promise");

const connection = await mysql.createConnection({
  host: "localhost",
  user: "username",
  password: "password",
  database: "mydb",
});

// Query
const [rows] = await connection.execute("SELECT * FROM users WHERE id = ?", [
  1,
]);
```

#### MongoDB (Node.js with mongodb)

```javascript
const { MongoClient } = require("mongodb");

const client = new MongoClient("mongodb://localhost:27017");
await client.connect();

const db = client.db("mydb");
const users = db.collection("users");

// Query
const user = await users.findOne({ _id: 1 });
```

## CRUD Operations Comparison

### Create (Insert)

#### PostgreSQL/MySQL

```sql
INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com');
```

#### MongoDB

```javascript
await users.insertOne({ name: "John Doe", email: "john@example.com" });
```

### Read (Select)

#### PostgreSQL/MySQL

```sql
SELECT * FROM users WHERE email = 'john@example.com';
```

#### MongoDB

```javascript
await users.findOne({ email: "john@example.com" });
```

### Update

#### PostgreSQL/MySQL

```sql
UPDATE users SET name = 'Jane Doe' WHERE id = 1;
```

#### MongoDB

```javascript
await users.updateOne({ _id: 1 }, { $set: { name: "Jane Doe" } });
```

### Delete

#### PostgreSQL/MySQL

```sql
DELETE FROM users WHERE id = 1;
```

#### MongoDB

```javascript
await users.deleteOne({ _id: 1 });
```

## Best Practices

### Connection Pooling

Always use connection pooling for better performance:

```javascript
// PostgreSQL
const pool = new Pool({ max: 20 });

// MySQL
const pool = mysql.createPool({ connectionLimit: 10 });

// MongoDB
const client = new MongoClient(uri, { maxPoolSize: 10 });
```

### Indexing

Create indexes on frequently queried fields:

```sql
-- PostgreSQL/MySQL
CREATE INDEX idx_user_email ON users(email);
```

```javascript
// MongoDB
await users.createIndex({ email: 1 });
```

### Transactions

#### PostgreSQL/MySQL

```javascript
const connection = await pool.getConnection();
await connection.beginTransaction();

try {
  await connection.query("INSERT INTO users ...");
  await connection.query("INSERT INTO orders ...");
  await connection.commit();
} catch (error) {
  await connection.rollback();
  throw error;
}
```

#### MongoDB

```javascript
const session = client.startSession();
session.startTransaction();

try {
  await users.insertOne({ name: "John" }, { session });
  await orders.insertOne({ userId: 1 }, { session });
  await session.commitTransaction();
} catch (error) {
  await session.abortTransaction();
  throw error;
}
```

## Security

### SQL Injection Prevention

❌ **Bad** (SQL Injection vulnerable):

```javascript
const query = `SELECT * FROM users WHERE email = '${email}'`;
```

✅ **Good** (Using parameterized queries):

```javascript
const query = "SELECT * FROM users WHERE email = $1";
const result = await pool.query(query, [email]);
```

### MongoDB Injection Prevention

❌ **Bad**:

```javascript
await users.findOne({ email: req.body.email });
```

✅ **Good** (Validate input):

```javascript
const { email } = req.body;
if (typeof email !== "string") throw new Error("Invalid input");
await users.findOne({ email });
```

## Environment Variables

```bash
# .env
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
MYSQL_URL=mysql://user:password@localhost:3306/mydb
MONGODB_URL=mongodb://localhost:27017/mydb
```

## Docker Setup

See the [Docker section](../devops/docker/index.md) for containerized database setup.
