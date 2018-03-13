---
layout: post
title: "Deviseless, Headerless Authentication in Rails API"
categories: rails
excerpt: "Learn how to setup authentication in your API app where session info is to be returned in the response body rather than in the response header"
---

Here's the requirement:

1. You are to build a rails API application
2. The initial `/login` request will have `email` and `password`
3. After authenticating, the server must return a unique `session_id` (or some kind of access_token, depending on your use case) in the response body, not in the response header like how devise does
4. All subsequent requests for the remaining API endpoints must include `session_id` in the json request body, not in the request header like how devise does

How would you go about doing this?

Sure you can customize devise by overriding its session controller, or opt for devise token auth gem. Checkout the many ways [google shows you](https://encrypted.google.com/#q=devise%20return%20session%20in%20response).

But if you are building just an API app, devise is overkill. It comes with a lot of features, most of which focus on view files, emails etc.

Instead of resorting to a library, you can build a simple, but effective authentication system yourself. Rails provides all things necessary for it out of the box.

In this tutorial, I'm using Rails 5, but the lesson can be applied for older versions too, with some additional gem installs.

### The Steps
Your User model should have a field to store password. Since we'll use [`has_secure_password`](http://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html), we'll use the `password_digest` field for that.

The session_id field for each user will have to be unique for all users. Only based on this we'll be able to find the logged in user. Each login request should be responded with unique session_id. We can use rails 5's new [`has_secure_token`](https://github.com/robertomiranda/has_secure_token/blob/master/lib/has_secure_token.rb) for this.

Once this is done, with a user record present already, we can login like so:

```rb
  # some controller
  def login
    user = User.find_by username: params[:username]
    if user.authenticate(params[:password])
      user.regenerate_session_id
      @session_id = user.session_id
    else
      # inform login failure
    end
  end
```

The `user.authenticate(password)` method is provided by `has_secure_password`. You know what it does.

The `user.regenerate_session_id` method is provided by `has_secure_token`. Calling it each time updates the `session_id` field with new unique token each time. We'd want that because each successful login should generate a new session_id. Old session_ids shouldn't be possible to use.


### Conclusion
The excellent Railscasts video on building [authentication from scracth](http://railscasts.com/episodes/250-authentication-from-scratch-revised) was instrumental in helping me implement this simple and effective solution.

Also, try to simplify things when you can. Devise need not be the de-facto gem for your authentication needs.
