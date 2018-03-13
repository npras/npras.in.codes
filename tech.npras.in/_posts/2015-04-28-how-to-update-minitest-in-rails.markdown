---
layout: post
title: How to Update Minitest in Rails
categories: rails
excerpt: rails and minitest
---

The latest minitest version 5.6.0 has introduced a new way to assert expectations. In prior versions, the spec expectation methods like `must_equal`, `must_match` etc were called directly on the object being tested.

<pre class="lang:ruby decode:true ">(1 + 1).must_equal 2</pre>

Although this enhances readability and follows the general spec pattern introduced by rspec, this is achieved by modifying the system under test (SUT) itself. The expectations are methods added to the `Object` class which is one of the ancestor object of all ruby objects. This means tampering. Although the truth is, this doesn't really get in the way of the SUT, some people raised concerns. And so, a new wrapper object was introduced from minitest 5.6.0\. The expectations are no more monkey-patched into a high-level ruby class, instead they now belong to only this class- `MiniTest::Expectations`. So now, the assertions can be written as:

<pre class="lang:ruby decode:true">_(1 + 1).must_equal 2</pre>

The additional reason to bother about this new change is that, this will become the only way to assert in minitest in the coming versions. The current method will be deprecated some time in the future ([http://www.rubydoc.info/gems/minitest/5.6.0/Minitest/Expectations#value-instance_method](http://www.rubydoc.info/gems/minitest/5.6.0/Minitest/Expectations#value-instance_method))

# Here's how to update minitest in your current rails app

Since the standard library version of minitest included in Ruby 2.2 is only on version 5.5.1, it needs to be updated by installing it as a Gemfile. First, add minitest to your Gemfile.

<pre class="lang:ruby decode:true">gem 'minitest'</pre>

Running `bundle install` now won't result in any gem installation as bundler sees that minitest is already being used from the ruby standard library. But to get it to the latest version, you need to bundle update it from the command-line:

<pre class="lang:sh decode:true">bundle update minitest</pre>

You might see that the new version is installed from the standard output message: "Installing minitest 5.6.0 (was 5.5.1)" You can also ensure that your rails app is now using the new version by checking for it in the `rails console` with:

<pre class="lang:ruby decode:true ">MiniTest::Unit::VERSION</pre>

which will now return "5.6.0". Now you can change all your tests to use expectations and they'll pass and be future proof.
