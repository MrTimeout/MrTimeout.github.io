---
title: Redis String Data Type
author:
  name: MrTimeout
  link: https://github.com/MrTimeout
date: 2019-08-09 20:55:00 +0800
categories: [Database, Redis]
tags: [Redis, Database, String]
---

This post will guide you how to get your hands on Redis with String data type.

# String type

Strings just store sequences of bytes, including text, serialized arrays and binary arrays.

So, how can we store them in redis database? Let's see

## Setting and getting strings

We can create a key with a string value in different ways:

```plaintext
# Create the key "greeting:1" with the value "Hello" inside of it.
> SET greeting:1 "Hello"
"OK"

# Retrieve the value stored inside of the key
> GET greeting:1
"Hello"

# We can set multiple keys at the same time (being an atomic operation) instead of executing n "set" commands.
> MSET greeting:2 "Bye" greeting:3 "Good morning" greeting:4 "Good afternoon"
"OK"

# How many keys we have inserted?
> KEYS greeting:*
1) "greeting:4"
2) "greeting:3"
3) "greeting:2"
4) "greeting:1"

# That's OK, but I want its values
> MGET greeting:1 greeting:2 greeting:3 greeting:4 notexistent
1) "Hello"
2) "Bye"
3) "Good morning"
4) "Good afternoon"
5) "null"

# Same applies to get only one key string
> GET greeting:1
"Hello"

# We can also check if the variable exists before trying to set it to a value. 1 == True and 0 == False
> EXISTS greeting:5
(integer) 0
# It doesn't exists, so we will create one
> SET greeting:5 "Good night"
"OK"

# What could happen if while we check the existence of the key, other user tries to
# insert its own key? It can be a little messy, so Redis give us the option to check
# key and set if not existent in the same command, so it is thread-safe.
> SETNX greeting:6 "Hola"
(integer) 1

# If we execute the command anothe time. OK, the key is already set so we are fine.
> SETNX greeting:6 "Hola"
(integer) 0

# Ouch, I wrote the wrong value inside of greeting:6. I want to delete that key retrieving its value
> GETDEL greeting:6
"Hola"

# Check if it was deleted correctly. Perfect! It was.
> EXISTS greeting:6
(integer) 0
```

## Expiring string keys

What about expiring keys? We can set a time in seconds/milliseconds/unix-time-seconds/unix-time-milliseconds/keepttl to expire the key when it times out.

Time is expressed as:

- `EX <seconds>`: EXpire seconds
- `PX <milliseconds>`: eXPire milliseconds
- `EXAT <unix-time-seconds>`: EXpire AT unix-time-seconds
- `PXAT <unix-time-milliseconds>`: eXPire AT unix-time-milliseconds
- `KEEPTTL`: When modifying value of a key, we maintain the already set expire time

```plaintext
# Create a key string with expire time set to 2 hours. Here 1000 represents the User ID
> SET session:1000 "cookie" EX 7200
"OK"

# Create a key string with expire time set to 1 hour in milliseconds.
> SET session:1000 "cookie" PX 3600000
"OK"

# Update a key if it already exists and maintaining the expire time. We also return the previous
# stored value.
> SET session:1000 "cookie value" GET XX KEEPTTL
"cookie"

# Set a key if it doesn't exist with unix-time-seconds
# Current time  :    2022-08-18 12:40:00 (date -d "2022-08-18 12:40:00" +%s)
# EXpire AT time:    2022-08-18 12:45:00 (date -d "2022-08-18 12:45:00" +%s)
> SET session:1001 "another cookie value" NX EXAT 1660819500
"OK"

# After 5 minutes, we check if the key is still alive
> GET session:1001

# Set a key if it doesn't exist with unix-time-milliseconds
# Current time  :    2022-08-18 12:40:00 (date -d "2022-08-18 12:40:00" +%s)
# EXpire AT time:    2022-08-18 12:45:00 (date -d "2022-08-18 12:45:00" +%s)
> SET session:1002 "another cookie value" NX PXAT 1660819500000
"OK"

# After 5 minutes, we check if the key is still alive
> GET session:1002
```

## Commands

### Set

- `SET <key> <value> [NX | XX] [GET] [EX <seconds> | PX <milliseconds> | EXAT <unix-time-seconds> | PXAT <unix-time-milliseconds> | KEEPTTL]`:
  + NX: Not eXistent.
  + XX: EXXistent.
  + GET: GET the old value when updating and null when not existent.
  + EX: EXpire in seconds.
  + PX: eXPire in milliseconds.
  + EXAT: EXpire AT unix-time-seconds `date +%s`
  + PXAT: eXPire AT unix-time-milliseconds `echo $(($(date +%s) * 1000))`
  + KEEPTTL: keep the expire time set when updating value of the key. TTL -> Time To Live.
- `SETNX <key> <value>`: SET if Not eXistent. Similar to `SET <key> <value> NX`.
- `SETEX <key> <expire-time-seconds> <value>`: SET a key with value and EXpiration time in seconds. Similar to `SET <key> <value> EX <expire-time-seconds>`.
- `PSETEX <key> <expire-time-milliseconds> <value>`: SET a key with value and eXPiration time in milliseconds. Similar to `SET <key> <value> PX <expire-time-milliseconds>`.
- `MSET <key> <value> [<key> <value>...]`: SET Multiple keys with values in one command. This action is atomic (all keys are set at the same time).
- `MSETNX <key> <value> [<key> <value>...]`: SET Multiple keys with values in one command if they don't exists (the ones that already exist, are skipped). This action is atomic (all keys are set at the same time).

### Get

- `GET <key>`: It returns the value of the key and if it does not exists, it returns null
- `GETDEL <key>`: It returns the value of the key and deletes it.
- `GETEX <key> [EX <seconds> | PX <milliseconds> | EXAT <unix-time-seconds> | PXAT <unix-time-milliseconds> | PERSIST]`:
  + PERSIST: Remove the time to live associated with the key.
- `GETRANGE <key> <start> <end>`: being mykey "Hello"
  + `GETRANGE mykey 0 2` returns "Hel"
  + `GETRANGE mykey -2 -1` returns "ol
  + `GETRANGE mykey 0 10000` returns "Hello". There are not out of bounds.
- `GETSET <key> <value>`: Set a new value to a key and retrieve the old value. __Deprecated in Redis version 6.2.0__ Replaced with: `SET <key> <value> GET`

### Operations

- `APPEND <key> <value>`:
  + If key exists, it appends the value at the end of the actual value.
  + If key does not exists, it creates the key with the appended value.
- `LCS <key1> <key2> [LEN] [IDX] [MIMMATCHLEN len] [WITHMATCHLEN]`:
  + LCS means Longest Common Subsequence. This is not the same as LCS (Longest Common String), which means that characters do not to be contiguos.
  + LEN: To get the length of the string.
  + IDX: get the position of each string.
  + MIMMATCHLEN: Print only the ones that have the minimum "len" passed as a parameter.
  + WITHMATCHLEN: Print also the length of each match.

## Reference

- [STRING](https://redis.io/commands/?group=string)
- [SET](https://redis.io/commands/set/)
- [SETNX](https://redis.io/commands/setnx/)
- [SETEX](https://redis.io/commands/setex/)
- [PSETEX](https://redis.io/commands/psetex/)
- [MSET](https://redis.io/commands/mset/)
- [MSETNX](https://redis.io/commands/msetnx/)
- [GET](https://redis.io/commands/get/)
- [GETDEL](https://redis.io/commands/getdel/)
- [GETEX](https://redis.io/commands/getex/)
- [GETRANGE](https://redis.io/commands/getrange/)
- [GETSET](https://redis.io/commands/getset/)
- [APPEND](https://redis.io/commands/append/)