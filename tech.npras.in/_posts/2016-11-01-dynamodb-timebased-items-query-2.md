---
layout: post
title: "DynamoDB: Fetching Latest Items By Time: Part 2: Scan and Query APIs coupled with some Logic"
categories: aws
excerpt: "Learn how to fetch latest items in DynamoDB that's dependent on the Primary Keys"
---

Once the items are created in DynamoDB table using the partition keys we designed [**previously**]({% post_url 2016-10-31-dynamodb-timebased-items-query-1 %}), using the following strategy, we can fetch the latest items since the last run.

Remember that, in the last post, I mentioned that we'll be saving the details of the last saved item in the last run, in a file.

If that file is not present, the ReportScript will assume it is being run for the very first time. At that time, we don't have any partition or sort keys to act as search keys. So we'll use the Scan API to get all items from the beginning.

```rb
def get_items_without_filter
  opts = { table_name: 'my_ddb_table', limit: 1000 }
  get_items_by(:scan, opts)
end

def get_items_by(method, opts)
  items = []
  next_key = nil
  loop do
    opts = opts.merge(exclusive_start_key: next_key)
    result = @client.send(method, opts)
    items.push(*result.items)
    break if result.last_evaluated_key.nil?
    next_key = result.last_evaluated_key
  end
  items
end

```

In the subsequent runs, with the details of the last item saved, we can use the [Query API](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_Query.html) to get the latest items. The Query API needs a mandaroty parition key. Here's where our choice of using the hour quarters along with the timestamp will help us.

If the last saved item's `quarter_hour_timestamp` is "201601010100" (1 AM at Jan 1), and if the current time is 2:35 PM at Jan 1, then using simple calculation, we can find all the quarters between these 2 times. We want the hour quaters between 1 AM and 2:35 PM. They will be: "2016010101**15**", "2016010101**30**", "2016010101**45**", "2016010102**00**", "2016010102**15**", "2016010102**30**".

Here's the code that does this calculation:

```rb
def quarter_hours_array(last_hour, current_hour)
  regex = /
  (?<year>\d{4})
  (?<month>\d{2})
  (?<day>\d{2})
  (?<hour>\d{2})
  /x

  md = last_hour.match(regex)

  md_current_hour = current_hour.match(regex)
  current_hour = DateTime.new(md_current_hour[:year].to_i, md_current_hour[:month].to_i, md_current_hour[:day].to_i, md_current_hour[:hour].to_i)

  last_hour_date = DateTime.new(md[:year].to_i, md[:month].to_i, md[:day].to_i, md[:hour].to_i)
  current_date = current_hour ? current_hour : DateTime.now

  hour_step = (1.to_f)/24
  hour_quarters = ['00', '15', '30', '45']

  result = last_hour_date.step(current_date, hour_step).map do |date|
    year = date.strftime "%Y"
    month = date.strftime "%m"
    day = date.strftime "%d"
    hour = date.strftime "%H"
    hour_quarters.map { |quarter| "#{year}#{month}#{day}#{hour}#{quarter}" }
  end.flatten
  result.select{|quarter| quarter >= last_hour}
end
```

Now with this calculation piece of code in place, we can use to get the quarters between the last item's time and the current time, and using the Query API, we can loop over each quarter, send a single Query API for that quarter, and aggregate the results in an array.

Here's the code that does that:

```rb
def get_items_with_filter(filter)
  items = []
  opts = { table_name: 'my_ddb_table', limit: 1000 }
  condition = "timestamp_hour_quarter = :ts_hour_quarter AND timestamp_random > :ts_random"
  opts = opts.merge(
    key_condition_expression: condition,
    expression_attribute_values: {
      ':ts_hour_quarter' => nil,
      ':ts_random' => filter[:random_timestamps][0].split('.')[0], # ignoring the random string part. It works!
    },
  )
  quarter_timestamps = ts_hours_till_now(filter[:timestamp_hour_quarter], nil)
  quarter_timestamps.each do |quarter_timestamp|
    opts[:expression_attribute_values][':ts_hour_quarter'] = quarter_timestamp
    items_from_api = get_items_by(:query, opts)
    # this is to ensure there's no duplicate entries
    items_from_api.delete_if do |item|
      filter[:random_timestamps].include?(item['timestamp_random'])
    end
    items.push(*items_from_api)
  end
  items
end
```
