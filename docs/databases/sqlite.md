# SQlite

SQLite is a lightweight, file-based database that stores all data in a single file instead of running as a separate database server.

Think of it as **A database you can carry around as a file**

Simple explanation

Most databases work like this:

`App → Database Server → Data`

SQLite works like this:

`App → Database File (.db)`

![Traditional Database vs SQlite](../assets/images/database/traditional-vs-sqlite.png)
<b>No server. No setup. No username/password.</b>

## Key characteristics of SQLite

### 1️⃣ Serverless

- No database service to run

* No background process

- Your app directly reads/writes the file

### 2️⃣ File-based

- Entire database lives in one file (e.g. dev.db)
- Easy to copy, move, backup, or delete

### 3️⃣ Zero configuration

- No installation needed
- Works out of the box
- Perfect for beginners and local development

### 4️⃣ Very fast for small apps

**Great performance for:**

- Small to medium datasets
- Single-user or low-concurrency apps

## Where SQLite is commonly used

1. Mobile apps (Android, iOS)
2. Desktop apps
3. Browsers (Chrome, Firefox internally use SQLite)
4. Local development for web apps
5. Prototypes & demos
