---
layout: post
title: "DynamoDB: Fetching Latest Items By Time: Part 1: Choosing the right Primary Keys"
categories: aws
excerpt: "Wisely choose the range key and sort key depending on your needs"
---

I worked recently on a client project that had this DynamoDB requirement: The main app (rails) will send important billing related transactions as items to DynamoDB. For example, if a customer bought a specific agency's product, then the details of the purchase will be recorded so that the agency can be billed later on (to charge a commission). Since the ecosystem surrounding this app is designed to cater millions of users, the app that creates the Data Records in DynamoDB is not loaded with an additional purpose of handling the billing too. That part is handled by a separate script that runs via cron. That script - the ReportScript - will pull all latest items from DynamoDB, and create reports and saves them in the cloud on each run (it runs about every 1 hour).

The main challenge we faced was with identifying the proper primary keys. This should facilitate the easy fetching of records from the ReportScript. What made it even more an interesting problem was that the client wished to configure the cron interval as he wishes. So he can set it to run every 1 hour or every 15 minutes (at the very least). This means the ReportScript should be able to optimally use the API calls to fetch latest items.


## The Initial Suboptimal Strategy
The initial idea we ran with was this: Use the timestamp as the partition key, and the agency_id as the sort key.

This would work from the app's side. The rails app can send the data records, stamping the current time on each item, and sending all other details along with it. But it created a problem at the ReportScript side. We used the DynamoDB's [Scan API](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_Scan.html) to get the initial set of records, and then again the Scan API to get the latest items since the last run.

A small detour: After each run, we save the details of the last saved item. This was used in the `FilterExpression` argument to filter out items 'earlier' than the search key.

When we tested this, it fared well in the very first run, but failed miserably in the eventual runs. Reason is, even from the second run, the Scan API does full table scan from the very beginning. It only applies the `FilterExpression` after it had fetched the items in a single call. This means that the ReportScript will take a lot more time than its previous run, every single run. This is less than suboptimal.


## The working Strategy
The solution to the problem is two folds.

* Choose the Primary Keys that suit your need (Our main need is querying the latest items from DynamoDB from ReportScript)
* Choose effective query strategy that takes advantage of the primary key choice we made above

We'll cover the first part in this post.


## Quaterly Timestamps and Random Timestamps

Here's the keys I came up with:

* `quarter_hour_timestamp` - partition key. Eg: '201610230130'
* `random_timestamp` - sort key. Eg: '2016-10-23T01:30:00.7ZkKN6dce92mgcDD'

The partition key is of the form: 'YYYYMMDDHHXX', where 'XX' is one of the these - '00', '15', '30', '45', each representing a quarter of a hour. This simply means that items created within 15 minutes (that fall within 1 of these defined quarters) will end up in a single partition. The main advantage of this choice will be seen when we fetch items in the ReportScript app (which will be discussed in the next post).

The sort key is a timestamp suffixed with a random string generated from the rails app. The random string part of the this key ensures that the combination of the partition key and sort key is always unique for all items in the DynamoDB table.

Detour: If you are using rails/ruby, you can use the the SecureRandom library to generate this random string: `SecureRandom.base58(16)`

In ruby, you can construct both of these keys like so:

* `quarter_hour_timestamp`

```rb
time_now = Time.now.utc
rounded_time_now = time_now - time_now.sec - (time_now.min % 15) * 60
quarter_hour_timestamp = rounded_time_now.strftime("%Y%m%d%H%M")
```

* `random_timestamp`

```rb
time_now = Time.now.utc.strftime("%Y-%m-%dT%H:%M:%S")
random_string = SecureRandom.base58(16)
random_timestamp = "#{time_now}.#{random_string}"
```

Creating a table with these keys will allow you to fetch items based on their created time. You'll be able to easily fetch all latest items since the last run effectively. We'll see how, in the next post.


## Outro
Checkout the [**next post**]({% post_url 2016-11-01-dynamodb-timebased-items-query-2 %}) that details how these primary keys are taken advantage of to fetch latest items based on time.
