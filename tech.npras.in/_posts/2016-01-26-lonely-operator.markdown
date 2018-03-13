---
layout: post
title: The new Lonely Operator from Ruby 2.3
categories: code
excerpt: Or the Safe-navigation operator
---

Consider this extremely contrived example:

```rb
class Dummy
  def hello
  'hi hello'
  end
end
```

Assume that you need to access the hello method's value for some reason. The steps required would be:
* instantiate Dummy object, that is if the object creation was successful
* call the hello method on the object, that is if there's indeed a method called 'hello'
* call upcase on the result of hello method, that is if the 'hello' method returns a string.

So to do this, we write a very defensive code like this:

```rb
d = Dummy.new
if d && d.hello && d.hello.upcase == 'HI HELLO'
  p 'in if....'
else
  p 'in else....'
end
```

This is not fine and dandy. The code practically reads like the coder's horoscope. It says its coder treads even a text editor like he would tread a snake infested floor. Or to put it mildly, like Avdi Grimm, this code's coder is not confident.

To make coder's more confident, rails came up with a try. That is, literally a try.

```rb
require 'active_support'
require 'active_support/core_ext'

if d.try(:hello).try(:upcase) == 'HI HELLO'
  p 'in if....'
else
  p 'in else....'
end
```

Just by extending rails's ActiveSupport's Core Extensions, we get to be more confident. In case when any of the intermediary terms in the chain becomes nil, then the code won't blow in our face. It will just return nil, which would just go to the else branch of the conditional.

Hmm. Doesn't sound that confident to me considering that we 'require' activesupport to our aid every time we are faced with this situation. Makes me look like a bullied kid who brought his elder brother as a reinforcement next day to school.

How else do we write this code more confidently?

By being lonely and contemplating the dot.

Yes. Literally.

Ruby 2.3 introduces the lonely operator '&amp;.' which makes you write the above logic like so:

<pre class="lang:ruby decode:true ">if d&amp;.hello&amp;.upcase == 'HI HELLO'
  p 'in if....'
else
  p 'in else....'
end</pre>
&nbsp;

If the intermediary values are nil, then this will silently return nil, just as try would.

But it also does one thing better than try. If the method 'hello' is mispelled, then try would still silently return nil. It won't throw an error pointing to the typo. The lonely operator will. Let us 'undefine' the hello method and try the conditional again.

&nbsp;
<pre class="lang:ruby decode:true ">class Dummy
  undef hello
end

if d&amp;.hello&amp;.upcase == 'HI HELLO'
  p 'in if....'
else
  p 'in else....'
end</pre>
&nbsp;

The lonely operator will correctly tell us now that there's no method called 'hello' for Dummy.
<pre class="lang:default decode:true ">undefined method `hello' for #&lt;Dummy:0x007ff469177ba8&gt; (NoMethodError)

</pre>
&nbsp;

So now, you don't have to resort activerecord's try. It's implemented in a much better way as a native ruby behaviour. One more great reason to use ruby 2.3.
