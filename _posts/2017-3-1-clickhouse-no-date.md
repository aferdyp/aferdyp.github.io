---
layout: post
comments: false
title:  "ClickHouse - MergeTree with no date column"
date:   2017-03-01 00:00:00 +0000
categories: ClickHouse
---

I ran into a scenario where the table I needed to work with had no date column. While the [ClickHouse documentation](https://clickhouse.yandex/reference_en.html) is pretty good, it took me a little while to figure out handling tables with no date columns.

I had a LOG table (created with ENGINE=LOG).

    :) desc profile;

    DESCRIBE TABLE profile

    ┌─name───────┬─type───────────┬─default_type─┬─default_expression─┐
    │ age        │ UInt8          │              │                    │
    │ occupation │ String         │              │                    │
    │ gender     │ FixedString(1) │              │                    │
    │ salary     │ UInt32         │              │                    │
    └────────────┴────────────────┴──────────────┴────────────────────┘
    
I wanted to convert this in to a MergeTree table. Since I wasn't sure of what to do in this scenario, I tried doing something like so -

    CREATE TABLE profile_mt ENGINE = MergeTree((age, occupation, gender, salary), 8192) AS
    SELECT 
        age, 
        occupation, 
        gender, 
        salary
    FROM profile 

This was wrong and ClickHouse complained. It spewed out a long error message and somewhere in there was a clue to what was needed to be done.

    Received exception from server:
    Code: 42. DB::Exception: Received from localhost:9000, 127.0.0.1. DB::Exception: Storage MergeTree requires 3 or 4 parameters: 
    name of column with date,
    [sampling element of primary key],
    primary key expression,
    index granularity
    
    ...
    
    If your source data doesn't have any date or time, you may just pass any constant for date column while loading.
    
    ...

This "DDL" worked.

    CREATE TABLE profile_mt ENGINE = MergeTree(const_date, (age, occupation, gender, salary), 8192) AS
    SELECT 
        toDate('1970-01-01') AS const_date, 
        age, 
        occupation, 
        gender, 
        salary
    FROM profile 

PS - Something I did notice was that the date '1970-01-01' was treated as '0000-00-00'. However, a date of '2000-01-01' was treated as is. Since I wasn't trying to do too much with dates, it did not matter to me.


