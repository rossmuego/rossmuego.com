---
layout: post
title: 'Weblog - February'
date: 2019-02-28
excerpt: 'Blog entry for February.'
tag:
  - blog
  - weblog
  - placement
---

# February

The start of the year began with a team reshuffle, from 2 larger teams down to smaller teams. Team sizes are now around 3/4, with each team having a specific focus for each sprint/release. My team now consists of me plus 3 others (2 developers and a tester) and our main focus just now is on a new feature to be delivered, Global Reporting. My team will be working alongside another team of 4 to be implementing the new feature.

## The Work

The majority of the work that I have been doing this month has been due to the Global Reporting feature that my team is partly responsible for implementing. My job has been primarily to go through our existing codebase and change all of the `int` data types that I find to be `long` data types. This is due to how the new database structure will work for global reporting, whereas before we have multiple databases, each with a standard ID range, the idea is to move all of the records into a single database. This will require us to band the ID numbers together creating IDs larger than a regular `int` is capable of holding.

As well as just changing data types, a large part of the work has been to update our interfaces used in custom scripts to support these new ID ranges. Whereas it would be simple to just change the return datatype of a method to be `long` from `int`, this would be a breaking change as you cannot fit a `long` into an `int`. To get around this problem, I have created overload methods for the new data type, such as below;

### Previous code example

```c#
/// Public API method that can be called in scripts
public int GetIdNumber() {
  return 1_000_000;
}

// Custom code written in script
int newId = GetIdNumber();
```

### The problem

When changing the `GetIdNumber()` method to return a `long`, that would fail to compile as you cannot convert a `long` to an `int`.

This is the solution

```c#
public long GetIdNumber(){
  return 1_000_000_000_000;
}

public int GetIdNumber(){
  return 1_000_000;
}

int newId = GetIdNumber();
```

This way, none of the existing code is broken as the compiler can choose the correct method to use based on the signature.

### Other work

Some other bits of work I completed this sprint was an improvement to our internal diagnostic tools, by being able to see data ran against the database by the website in a grid.
