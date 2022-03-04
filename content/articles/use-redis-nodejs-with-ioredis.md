---
title: How to use Redis in NodeJS with ioredis
description: Quickly get started using Redis and NodeJS together
date: 2022-03-04T19:17:12.481Z
draft: false
lastmod: 2022-03-04T20:59:04.501Z
---

[Redis](https://redis.io/) is a blazingly fast in-memory NoSQL database. It works great for session storage, caching, and real-time applications that need incredibly fast response times.

This tutorial will help you get up and running with Redis and NodeJS quickly using `ioredis`.

In this tutorial you'll quickly get up and running connecting your NodeJS project to Redis. You'll also learn some basic Redis commands.

## Preparation

This tutorial assumes you already have a npm project and a Redis server set up. If you don't, check out the [extras section](#extras) at the end for instructions.

To get started, install `ioredis`:

```shell
npm install ioredis
```

## Setup

Now, you'll connect to the server. If Redis is running on `localhost` at the default port with no password (the defaults), then you don't need to provide any configuration.

```js
const Redis = require("ioredis");
const client = new Redis(); // No configuration needed
```

If Redis isn't using the defaults, pass in configuration when you create the client.

```js
const client = new Redis({
  port: 6379,
  host: "localhost",
  password: "your-password",
});
```

> You don't need to specify all the options, only those that aren't the defaults.
>
> ```js
> const client = new Redis({
>   port: 6380, // Uses all defaults, except for the port
> });
> ```

Now that you've connected to the server, it's time to send Redis commands!

## Usage

`ioredis` supports both NodeJS callbacks and promises for Redis commands.

```js
// NodeJS callbacks
client.get("key", (err, result) => {
  // handle if error...
  console.log(result);
});

// Promises (using await)
const result = await client.get("key");
console.log(result);

// or without await

client.get("key").then((result) => {
  console.log(result);
});
```

The rest of this tutorial will be written using promises and await, but use whichever style you're most comfortable with.

Now, you'll learn some basic Redis commands. These basic commands deal with key-value pairs, which are the most basic data structure in Redis.

### GET

`GET` retrieves a key from Redis, returning `null` if the key doesn't exist.

```js
const name = await client.get("name");
```

### SET

`SET` creates or updates a key in Redis, returning `"OK"` if successful, and `null` if unsuccessful.

```js
const result = await client.set("testkey", "testval");

// To only create the key if the key doesn't already exist
// (to not overwrite an existing key)
// use the NX (Not eXist) option
const result = await client.set("testkey", "testval", "NX");

console.log(result); // "OK" if 'testkey' doesn't exist, or null otherwise
```

### DEL

`DEL` deletes a key, returning the number of keys that were removed.

```js
const result = await client.del("testkey");
// 1 if `testkey` exists, or 0 if it doesn't
```

> This tutorial only covers the basics of Redis. For more information, check out [Common Redis commands on freeCodeCamp](https://www.freecodecamp.org/news/how-to-learn-redis/#common-redis-commands).

## Extras

These additional prerequisites might be needed if you don't already have things set up.

### Create a new npm project

Create a new npm project with `npm init` (make sure to run this in an empty folder you'll use for your project). Use `--yes` to skip the questions and use the defaults if you want to get up and running faster.

```shell
npm init --yes
```

### Install Redis

If you're running Ubuntu, there's an official PPA to use.

```shell
sudo add-apt-repository ppa:redislabs/redis
sudo apt update
sudo apt install redis-server
```

If you're running a different version of Linux, try using your default package manager (`apt`, `dnf`, or equivalent) or the snap package.

```shell
sudo snap install redis
```

If you're running Windows, check out [this tutorial on how to install Redis on Windows 10](https://developer.redis.com/create/windows/).

## Conclusion

You've learned how to use Redis with NodeJS. Now go build something amazing!
