---
layout: post
title: "Demonstrating Basic Probability with Ruby"
excerpt: "Out of 10 lovers, what's the probability that the 7th lover ends up with you?"
---

I'm not sure I can explain the following code I wrote clearly. But I'll try.

What's the probability that a specific number is drawn out of 10 numbers? Say '7' out of 1 to 10? It's 10% right?

If so, then on 10 repeated draws, at least once (or exactly once?) '7' should show up right? Is it guaranteed? Not always. It can be more or less than 10.

But how to __make sure__ it is close to the ideal 10% ?

We'll have to increase the number of the draws, aka the experiments. This code draws only once, through 10 crore times. The `variance` tells about how close the actual selection was to the ideal 10%.

The code demonstrates that, as the draws (or experiments) increase, the accuracy increases. The variance approaches 0 faster in higher values than in the initial low draw counts.

```ruby
dice_heads = 10
success_count = 0
selection = 7
ideal = 10.0

experiments = [
  { trials: 1, result: 0, variance: 0 },
  { trials: 10, result: 0, variance: 0 },
  { trials: 100, result: 0, variance: 0 },
  { trials: 1000, result: 0, variance: 0 },
  { trials: 10000, result: 0, variance: 0 },
  { trials: 100000, result: 0, variance: 0 },
  { trials: 1000000, result: 0, variance: 0 },
  { trials: 10000000, result: 0, variance: 0 },
  { trials: 100000000, result: 0, variance: 0 },
]


experiments.each do |experiment|
  experiment[:trials].times do
    success_count += 1 if rand(1..dice_heads) == selection
  end
  experiment[:result] = (success_count.to_f/experiment[:trials]) * 100
  experiment[:variance] = (ideal - experiment[:result]).abs.round(5)
  success_count = 0
end

puts experiments
```

Here are the results of the code:

```
{:trials=>1, :result=>0.0, :variant=>10.0}
{:trials=>10, :result=>20.0, :variant=>10.0}
{:trials=>100, :result=>11.0, :variant=>1.0}
{:trials=>1000, :result=>10.2, :variant=>0.2}
{:trials=>10000, :result=>10.040000000000001, :variant=>0.04}
{:trials=>100000, :result=>10.105, :variant=>0.105}
{:trials=>1000000, :result=>10.051400000000001, :variant=>0.0514}
{:trials=>10000000, :result=>9.97998, :variant=>0.02002}
{:trials=>100000000, :result=>9.992652, :variant=>0.00735}
```
