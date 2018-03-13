---
layout: post
title: "Reading from a file and writing to another file in Ruby"
excerpt: "Small snippet in idiomatic ruby"
---


```rb
def create_output_file
  CSV.open(@output_file, 'wb') { |f| yield(f) }
end
```

```rb
def the_work
  create_output_file do |out|
    csv_opts = { headers: true }
    headers = CSV.read(@input_file, headers: true).headers
    out << headers
    CSV.foreach(@input_file, csv_opts) { |row| out << processed_row(row) }
  end
end
```

```rb
def processed_row(row)
  # your complex row processing code
end
```
