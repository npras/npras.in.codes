---
layout: post
title: Exception Handlers only for Exceptional Cases
categories: ruby
excerpt: Reading books can help you make a better informed decision
---

Take a look at these 2 versions of the same method. They both try to generate a secret token that is has a unique index at database level.

Version 1 uses the db-raised exception as part of the code logic. *Mostly* it gets its job done in just 1 query - the update query.

Version 2 uses just a ruby loop until it finds a unique token. It doesn't use any exception, but requires a mininum of 2 queries to get the job done - the select and the udpate query.

Version 1:

```rb
def regenerate_secret_token
    secret_token = generate_unique_secure_secret_token
  update! secret_token: secret_token
rescue ActiveRecord::RecordNotUnique
  retry
end
```

Version 2:

```rb
def regenerate_secret_token
  loop do
    secret_token = generate_unique_secure_secret_token
    break secret_token unless User.where(secret_token: secret_token).exists?
  end
  update! secret_token: secret_token
end
```

Which one would you prefer?

I preferred Version1. The appealing factor was the 1 less db query.

### Enter Exceptional Ruby

[The book](http://exceptionalruby.com/) said this about when to use exceptions really:

> Will this code still run if I remove all the exception handlers?” If the answer is “no”, then maybe exceptions are being used in non-exceptional circumstances.

Version1 won't run if I remove the exception handler code. In other words, the exception handler doesn't handle truly "exceptional" cases. It just handles normal expected cases. So, this handler should be part of the method's body, not in the exception handler section.

Version2 FTW.
