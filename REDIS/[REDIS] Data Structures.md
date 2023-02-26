## Lists

Redis Lists are implemented with *linked lists* because for a database system it is crucial to be able to add elements to a very long list in a very fast way.

### O(1) 

- `LPUSH`, `RPUSH`
- `LPOP`, `RPOP`
- `LLEN`

<br>

### O(n)

- `LINDEX`, `LRANGE`
- `LSET`

<br>

### Use Case

Feed system can be a good use case. 

Every time a user posts a new photo, we add its *ID* into a list with `LPUSH`. When users visit the home page, we use `LRANGE 0 9` in order to get the latest 10 posted items.

<br>

## Sets

Sets stores collections of unique, unsorted *string elements*.

Add, remove, and check for the existence of members is very cheap.

If you have a collection of items and it is very important to check for the existence(`SISMEMBER`) or size of the collection(`SCARD`) in a very fast way.

Another cool thing about sets is support for peeking(`SRANDMEMBER`) or popping random elements(`SPOP`).

<br>

### O(1)

|TYPE|COMMAND|
|-|-|
|CREATE|`SADD`|
|DELETE|`SPOP`|
|RETRIEVE|`SISMEMBER`|
|OTHERS|[`SCARD`](https://redis.io/commands/scard/)|

<br>

### O(N)

|TYPE|COMMAND|
|-|-|
|DELETE|[`SREM`](https://redis.io/commands/srem/)|
|RETRIEVE|[`SDIFF`](https://redis.io/commands/sdiff/)|
|OTHERS|`SUNION`|
|LIST|`SMEMBERS`|

<br>

### O(N*M)

|TYPE|COMMAND|
|-|-|
|OTHERS|[`SINTER`](https://redis.io/commands/sinter/)|

<br>

### Use Case

**Sets are good for expressing relations between objects.**

For example, we can use sets in order to implement *many-to-many relationships* between posts and tags.

<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*MhT1Jn1VN9d_UiFfXvIb9w.png">

> 출처: https://medium.com/analytics-vidhya/the-most-important-redis-data-structures-you-must-understand-2e95b5cf2bce

**1. To get all the posts tagged in by MYSQL**

```
smembers tag:Mysql:posts
```

**2. To get all the posts tagged by multiple tags like MYSQL, Java, Redis.**

```
sinter tag:Java:posts tag:MySQL:posts, tag:Redis:posts
```

<br>

## Hash

Redis Hashes can be used to represent objects.

<br>

### O(1)

|TYPE|COMMAND|
|-|-|
|CREATE|`HSET`|
|RETRIEVE|`HGET`|
|OTHERS|`HLEN`|

<br>

### O(n)

|TYPE|COMMAND|
|-|-|
|RETRIEVE|`HGETALL`|
|LIST|`HKEYS`|

<br>

### Use Case

We can use Hash map to model a user(whatever can be a model).

<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*2L5ORp-8KMmM5ChTR3h94A.png">

```
HMSET user:139960061 login dr_josiah id 139960061 followers 176
OK

HGETALL user:139960061
1) "login"
2) "dr_josiah"
3) "id"
4) "139960061"
5) "followers"
6) "176"

HINCRBY user:139960061 followers 1
"177"
```

<br>

## Sorted Sets

Every member of a Sorted Set is associated with a score, that is used in order to take the sorted set ordered, from the smallest to the greatest score.

> *... Accessing the middle of a sorted set is also very fast, so you can use Sorted Sets as **a smart list of non repeating elements** where you can quickly access everything you need: elements in order, fast existence test, fast access to elements in the middle!*

<br>

### O(1)

|TYPE|COMMAND|
|-|-|
|OTHERS|`ZCARD`|

<br>

### O(log(n))

|TYPE|COMMAND|
|-|-|
|CREATE|`ZADD`|
|DELETE|`ZREM`, `ZPOPMAX`|
|RETRIEVE|`ZRANGE`, `ZRANK`|

<br>

### Use Case

Many Q&A platforms like StackOverflow and Quara use Redis Sorted Sets to rank the highest voted answers for each proposed question to ensure the best content is listed at the top of the page.