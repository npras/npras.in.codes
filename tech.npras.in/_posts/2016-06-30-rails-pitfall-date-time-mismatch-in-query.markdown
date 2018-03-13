---
layout: post
title: "Rails Pitfall: Date, Time mismatch in Query"
categories: rails
excerpt: Passing Date object to where DateTime object is expected will bite you in the back
---

Often times you'd have to query a table for set of records that fall within a given date range.

Example: In your admin interface, you'd want to see all the new customers that came to your product in the month of Jan.

You'll provide a date range selector in your view. You'll select `from` and `to` dates as Jan 1, 2016 and Jan 31, 2016.

To your controller, these values might come as strings: `"2016-01-01"` and `"2016-01-31"`

And you pass it to this query like so:

```rb
from_date = params[:from_date].to_date
to_date = params[:to_date].to_date
User.where(created_at: (from_date..to_date))
```

Looks correct right?

Look again.

`from_date` and `to_date` are `Date` objects. But `created_at` is a `datetime` type in the database. So when you pass date objects where datetime objects are expected, instead of raising hell, `ActiveRecord` just converts it into a time object and proceeds to execute the query.

I'm not sure how it converts, but it might be as simple as this:

```rb
str_date = "2016-01-31"
date = str_date.to_date # => Sun, 31 Jan 2016
date.class # Date
date_with_time = date.to_time # => 2016-01-31 00:00:00 +0530
date_with_time.class # => Time
# or even:
date_with_time = date.to_datetime # => Sun, 31 Jan 2016 00:00:00 +0000
date_with_time.class # => DateTime
```

Whether it is `Time` or `DateTime`, note that the time part of the date is `00:00:00`. That's the **very start of the day** - at 12AM.

**It doesn't not count the rest of the day**.

So if there were 5 users created on Jan 31, they will not be included in the above query.

So make sure to pass time object to your query always because your DB mostly speaks those fields only

```rb
User.where(created_at: (from_date.beginning_of_day..to_date.end_of_day))
```

This will convert the date objects to `ActiveSupport::TimeWithZone`.

```rb
date.end_of_day # Sun, 31 Jan 2016 23:59:59 UTC +00:00
date.end_of_day.class # ActiveSupport::TimeWithZone
```

Note the time fields now: `23:59:59`. Your query will now include the entire of Jan 31, and you'll not miss any result.
