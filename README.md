# Cotton

![ci](https://github.com/rahmanfadhil/cotton/workflows/ci/badge.svg?branch=master) ![GitHub release (latest by date)](https://img.shields.io/github/v/release/rahmanfadhil/cotton)

**SQL Database Toolkit for Deno.**

- Well-tested
- Type-safe
- Supports **MySQL**, **SQLite**, and **PostgreSQL**
- Semantic versioning

## Documentation

- [Getting started guide](https://rahmanfadhil.github.io/cotton)
- [Explore the API](https://doc.deno.land/https/deno.land/x/cotton/mod.ts)

## How to use

Currently, Cotton supports [SQLite3](https://sqlite.org), [MySQL](https://mysql.com), and [PostgreSQL](https://postgresql.org). To create a connection, use `connect` and pass the connection configurations.

```ts
import { connect } from "https://deno.land/x/cotton/mod.ts";

const db = await connect({
  type: "sqlite", // available type: 'mysql', 'postgres', and 'sqlite'
  database: "db.sqlite",
  // other...
});
```

You can run an SQL statement using the `execute` method.

```ts
await db.query(`
  CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email VARCHAR(255),
  );
`);
```

Cotton provides an easy-to-use query builder which allows you to perform queries without writing raw SQL.

```ts
// Execute "SELECT * FROM users;"
const users = await db.table("users").execute();

for (const user in users) {
  console.log(user.email);
}
```

However, you can still use raw SQL via `query` method.

```ts
const users = await db.query("SELECT * FROM users;");

for (const user of users) {
  console.log(user.email);
}
```

Once, you've finished using the database, disconnect it.

```ts
await db.disconnect();
```

## Model

A model is nothing more than a class that extends `Model`.

```ts
import { Model } from "https://deno.land/x/cotton/mod.ts";

class User extends Model {
  static tableName = "users";

  @Field()
  email!: string;

  @Field()
  age!: number;

  @Field()
  created_at!: Date;
}
```

Keep in mind that you need to override the default TypeScript configration in order to use this decorator feature.

```json
// tsconfig.json

{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

```
$ deno run -c tsconfig.json main.ts
```

To do CRUD operations to our model, we can use the provided method in our model:

```ts
const user = await User.findOne(1); // find user by id
console.log(user instanceof User); // true
```

```ts
const users = await User.find(); // find all users

for (const user in users) {
  console.log(user.email);
}
```

To save the current model to the database, use the `save` method.

```ts
const user = new User();
user.email = "a@b.com";
user.age = 16;
user.created_at = new Date("1 June, 2020");
await user.save();
```

You also can use the `insert` method to create the model instance and save it to the database at the same time.

```ts
const user = await User.insert({
  email: "a@b.com",
  age: 16,
  created_at: new Date("1 June, 2020"),
});
```

To insert multiple records, you can simply pass an array as the parameter.

```ts
const user = await User.insert([
  { email: "a@b.com", age: 16, created_at: new Date("1 June, 2020") },
  { email: "b@c.com", age: 17, created_at: new Date("2 June, 2020") },
]);
```

## Query Builder

### Basic query

```ts
await db
  .table("users")
  .where("email", "a@b.com")
  .where("name", "john")
  .execute();
// SELECT * FROM users WHERE email = 'a@b.com' AND name = 'john';
```

### orWhere and notWhere

```ts
await db.table("users").notWhere("name", "kevin").execute();
// SELECT * FROM users WHERE NOT name = 'kevin';

await db
  .table("users")
  .where("name", "kevin")
  .orWhere("name", "john")
  .execute();
// SELECT * FROM users WHERE name = 'kevin' OR name = 'john';
```

### Select columns

```ts
await db.table("users").select("email").execute();
// SELECT (email) FROM users;

await db.table("users").select("id", "email").execute();
// SELECT (id, email) FROM users;

await db.table("users").select("id").select("email").execute();
// SELECT (id, email) FROM users;
```

### Pagination

```ts
await db.table("users").limit(5).offset(5).execute(); // Skip 5 row and take 5
// SELECT * FROM users LIMIT 5 OFFSET 5;
```

### Insert data

```ts
await db
  .table("users")
  .insert({
    email: "a@b.com",
    age: 16,
    created_at: new Date("5 June, 2020"),
  })
  .execute();
// INSERT INTO users (email, age, created_at) VALUES ('a@b.com', 16, '2020-06-05 00:00:00');

await db
  .table("users")
  .insert([{ email: "a@b.com" }, { email: "b@c.com" }])
  .execute();
// INSERT INTO users (email) VALUES ('a@b.com'), ('a@b.com');
```

### Replace data

```ts
await db
  .table("users")
  .replace({
    email: "a@b.com",
    age: 16,
    created_at: new Date("5 June, 2020"),
  })
  .execute();
// REPLACE INTO users (email, age, created_at) VALUES ('a@b.com', 16, '2020-06-05 00:00:00');
```

### Delete data

```ts
await db.table("users").where("email", "a@b.com").delete().execute();
// DELETE FROM users WHERE email = 'a@b.com';
```

### Update data

```ts
await db
  .table("users")
  .where("email", "a@b.com")
  .update({ name: "John" })
  .execute();
// UPDATE users SET name = 'John' WHERE email = 'a@b.com';
```
