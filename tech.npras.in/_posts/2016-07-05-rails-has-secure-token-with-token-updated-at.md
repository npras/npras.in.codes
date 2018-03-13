---
layout: post
title: "Rails has_secure_token with token_updated_at"
categories: rails
excerpt: customising has_secure_token to include token expiration field
---

Rails comes with the awesome [has_secure_token](http://api.rubyonrails.org/classes/ActiveRecord/SecureToken/ClassMethods.html).

It comes in very handy when you need a unique *and* secure token. It's most common usecase is to be used in places where an authentication token is sent from the server.

Calling this class method gives your model object a useful method: `regenerate_token` (that's if your token field's name in db is "token").

Using this method, our model object will automatically sql-updated with a new secure token.

All fine so far.

But here's a usecase: What if you are using this token for authentication purpose and you need to invalidate the token after, say, 24 hours. How would you go about doing that efficiently?

You'd need a `token_updated_at` timestamp field in your model because you can't just rely on `updated_at` alone. It could trigger false results.

When `regenerate_token` is being called, along with the `token`, we also want this new field to be updated as well with latest timestamp.

You can do it in a separate query, but by this time, if you are using `has_secure_password` too, then the number of queries will easily start to increase - all fetching the same record again and again.

`regenerate_token` doesn't provide a way to extend it.

But we can **not** use it altogether and implement the logic ourself! The code for it is very simple. [Check it out](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/secure_token.rb).

So after removing the `has_secure_token` call, and leaving the `token` and `token_updated_at` columns as such, we'll have to implement the `regenerate_token` ourselves:

```rb
# In your user.rb
  def regenerate_token
    loop do
      token = generate_unique_secure_token
      break token unless User.where(token: token).exists?
    end
    update! token: token, token_updated_at: Time.now   
  end

  def generate_unique_secure_token
    SecureRandom.base58(24)
  end
```

Now, in a single query both the fields will be updated.
