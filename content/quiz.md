---
title: "Quiz"
type: "page"
draft: False
---

Dump of quizes for new things I want to remember overtime 


### DDIA Quiz

{{< quizdown >}}

---
primary_color: '#9375fd'
secondary_color: lightgray
text_color: black
shuffle_questions: true
shuffleAnswers: true
locale: en
---

## Advantages of using NoSQL in large scale distributed systems

---
shuffle_answers: true
---

why would you use NoSQL in large scale distributed systems over SQL databases?

> hint: 

- [X] NoSQL has better locality than SQL
- [X] NoSQL is easier to shard
    > This is complicated with a SQL database, because when using a join call, you
may potentially have to access many partitions resulting in lots of network calls. On the other hand, the increased locality of NoSQL means that the entire document will likely be on one machine and can be quickly accessed with a single network call
- [X] NoSQL data is not formatted
- [ ] NoSQL data is easier to query


## Advantages of using SQL databases in large scale distributed systems

why would you use SQL database in large scale distributed systems over NoSQL databases?

- [X] SQL allows joins, whereas NoSQL does not
    > SQL allows the data to be a bit more modular, where you can only request certain parts of a potentially large document, at the tradeoff that
    trying to fetch the entire document may take a long time
- [ ] SQL data is easier to shard
- [X] SQL has transactions
    > Caveat: in a distributed setting it rarely makes sense to use transactions and so this benefit is diminished
- [ ] SQL has data locality


## Downsides of using an in-memory hashmap as index for database

what prevents you from using an in-memory hashmap for database indexes?

- [X] Inefficient range queries since not all keys in a range can be scanned at once
- [ ] We cannot persist data in case of a crash or an outage
- [X] Many distinct keys are limited by the memory size


## Put the [days](https://en.wikipedia.org/wiki/Day) in order!

> Monday is the *first* day of the week.

1. Monday
2. Tuesday
3. Wednesday
4. Friday
5. Saturday  



{{< /quizdown >}}