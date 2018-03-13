---
layout: post
title: Using define_method in a Class to Create Class Methods and Instance Methods
categories: code
excerpt: too much metaruby in here
---

<a href="http://ruby-doc.org/core-2.2.2/Module.html#method-i-define_method">`define_method`</a> is one of the many powerful features of Ruby that makes possible the creation of awesome gems like Rails. Understanding its usage and how to wield it will make <em>you</em> awesome.

In this post, we'll see all the different ways we can use `define_method` to define class and instance methods in a Ruby class.

But before diving in, make sure you read <a href="http://yehudakatz.com/2009/11/15/metaprogramming-in-ruby-its-all-about-the-self/">Yehuda Katz's excellent post about `self`</a> (a pun here would be to 'Know Thyself!').

We'll need a class and an object for experimenting with.

```rb
class Dummy
end

dummy_object = Dummy.new
```

In order to make each examples stand on its own to enhance readability, we'll re-open our `Dummy` class as and when needed. We'll also use the same `dummy_object` object where necessary.
<h2>Simplest Usecase</h2>
<strong>Scenario 1)</strong> To define an instance method named `instance_hi` that takes a `name` argument and greets the name:

```rb
class Dummy
  define_method :instance_hi do |name|
    "hi '#{name}'"
  end
end

dummy_object = Dummy.new
puts dummy_object.instance_hi "instance method"
# =&gt; hi 'instance method'
```

<strong>Scenario 2)</strong> To define a class method named `class_hi`:
<pre class="lang:ruby decode:true">class Dummy
  class &lt;&lt; self
    # we are in Dummy's metaclass environment here
    define_method :class_hi do |name|
      "hi '#{name}'"
    end
  end
end

puts Dummy.class_hi "class method"
# =&gt; hi 'class method'</pre>
If you find `class &lt;&lt; self` tedious, and if your ruby version is &gt; 1.9.2, you can use <a href="http://ruby-doc.org/core-2.2.2/Object.html#method-i-define_singleton_method">`define_singleton_method`</a> like this:
<pre class="lang:default decode:true">class Dummy
  define_singleton_method :class_hi do |name|
    "hi '#{name}'"
  end
end

puts Dummy.class_hi "class method"
# =&gt; hi 'class method'</pre>
<strong>Note:</strong> The above examples define methods that only take arguments. How would you define methods that also use a block? Using scenario 1, here's how:
<pre class="lang:default decode:true">class Dummy
  define_method :instance_hi do |name, &amp;block|
    "hi '#{name}'#{block.call if block}"
  end
end

dummy_object = Dummy.new
puts dummy_object.instance_hi("instance method")
# =&gt; hi 'instance method'
puts dummy_object.instance_hi("instance method") { '!' }
# =&gt; hi 'instance method!</pre>
Note that you can't use `yield` to replace `block.call`. It would result in `LocalJumpError`.
<h2>Real-world Usecase</h2>
This is great to understand `define_method`. But it wasn't created to define a method name we already know! The real application for `define_method` is when we use it to define methods on the fly, dynamically.

We'll see how to create class and instance methods dynamically using `define_method` from existing class and instance methods.

<strong>Scenario 3) Defining an instance method from an existing instance method</strong>
<pre class="lang:ruby decode:true">class Dummy
  def create_instance_method_from_instance_method method_name, &amp;block
    self.class.send :define_method, method_name do |name, &amp;block|
      "hi '#{name}'#{block.call if block}"
    end
  end
end

dummy_object.create_instance_method_from_instance_method :instance_hi_from_instance_method
puts dummy_object.instance_hi_from_instance_method("dynamic instance method from instance method") { '!' }
# =&gt; hi 'dynamic instance method from instance method!'</pre>
Since `define_method` is a private method, it needs to be called via `send`.

<strong>Scenario 4) Defining a class method from an existing instance method</strong>

For all ruby versions:
<pre class="lang:ruby decode:true">class Dummy
  def create_class_method_from_instance_method method_name, &amp;block
    metaclass_of_dummy_class = (class &lt;&lt; self.class; self; end)
    metaclass_of_dummy_class.instance_eval do
      define_method method_name do |name, &amp;block|
        "hi '#{name}'#{block.call if block}"
      end
    end
  end
end

dummy_object.create_class_method_from_instance_method :class_hi_from_instance_method
puts Dummy.class_hi_from_instance_method('dynamic class method from instance method') { '!' }
# =&gt; hi 'dynamic class method from instance method!'</pre>
For ruby versions &gt; 1.9.2, using `define_singleton_method`:
<pre class="lang:ruby decode:true ">class Dummy
  def create_class_method_from_instance_method method_name, &amp;block
    self.class.define_singleton_method method_name do |name, &amp;block|
      "hi '#{name}'#{block.call if block}"
    end
  end
end

dummy_object.create_class_method_from_instance_method :class_hi_from_instance_method
puts Dummy.class_hi_from_instance_method('dynamic class method from instance method') { '!' }
# =&gt; hi 'dynamic class method from instance method'!</pre>
<strong>Scenario 5) Defining an instance method from an existing class method</strong>
<pre class="lang:ruby decode:true">class Dummy
  def self.create_instance_method_from_class_method method_name, &amp;block
    define_method method_name do |name, &amp;block|
      "hi '#{name}'#{block.call if block}"
    end
  end
end

Dummy.create_instance_method_from_class_method :instance_hi_from_class_method
puts dummy_object.instance_hi_from_class_method('dynamic instance method from class method') { '!' }
# =&gt; hi 'dynamic instance method from class method'!</pre>
<strong>Scenario 6) Defining a class method from an existing class method</strong>

For all ruby versions:
<pre class="lang:ruby decode:true ">class Dummy
  def self.create_class_method_from_class_method method_name, &amp;block
    metaclass_of_dummy_class = (class &lt;&lt; self; self; end)
    metaclass_of_dummy_class.instance_eval do
      define_method method_name do |name, &amp;block|
        "hi '#{name}'#{block.call if block}"
      end
    end
  end
end

Dummy.create_class_method_from_class_method :class_hi_from_class_method
puts Dummy.class_hi_from_class_method('dynamic class method from class method') { '!' }
# =&gt; hi 'dynamic class method from class method'!
</pre>
For ruby versions &gt; 1.9.2, using `define_singleton_method`:
<pre class="lang:ruby decode:true">class Dummy
  def self.create_class_method_from_class_method method_name, &amp;block
    define_singleton_method method_name do |name, &amp;block|
      "hi '#{name}'#{block.call if block}"
    end
  end
end

Dummy.create_class_method_from_class_method :class_hi_from_class_method
puts Dummy.class_hi_from_class_method('dynamic class method from class method') { '!' }
# =&gt; hi 'dynamic class method from class method'!</pre>
<h2>Conclusion</h2>
As you can see, understanding the value `self` holds in different environment (context) is crucial to understanding `define_method`.

Dynamic methods can also be defined using 'class_eval` and `instance_eval` too. But according to Aaron Patterson's research, it is slower compared to using `define_method`. You can read about the research, about how `class_eval` was used in rails, and one gotcha of using `define_method` in <a href="http://tenderlovemaking.com/2013/03/03/dynamic_method_definitions.html">this elaborate post </a>from him.
