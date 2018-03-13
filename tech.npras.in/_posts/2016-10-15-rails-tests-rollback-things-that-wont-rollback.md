---
layout: post
title: "Rails Tests: Rollback Things That Won't Rollback With Teardown"
excerpt: Learn why and how mocks have to be teared down
---

In each rails test case, the database events are rolled back after each run. This ensures the test database remains in a known consistent state before each testcase. We don't way any false positives (or negatives) due to the changes made by one test case affecting the others.

What won't be rolled back though, is any other non-database events that change the state of the app. It's very easy for us to forget to rollback them manually - either at the end of the testcase or in a teardown block. But getting into the habit of thinking in this line will save you a lot of headaches.

One such minor headache just turned itself to a blog post now.

Minitest provides a way to mock objects. You create a mock object using `Minitest::Mock.new`. You then set expectations on that object that some method(s) will be called with certain parameters and return certain other thing(s). And finally, after running the code you are testing, you verify if those expectations were met.

At work recently, I had write a test case where I had to mock an object in a class that talks to an external service via http. Mocking this particular object will allow me to not worry about faking the actual http request when the request details are not known (abstracted behind the SDK. hi AWS!). I can then set an expectation that this object will be called with a certain method that actually calls the external service.

Here's the simplified code:

```rb
  test "if the method talking to the external service is being called" do
    obj = objects(:obj1)
    refute obj.shiny?

    mock_obj = Minitest::Mock.new
    ExternalService.instance_variable_set :@service_obj, mock_obj
    mock_obj.expect :log, nil, [Hash]

    obj.shine!

    mock_obj.verify
  end
```

Somewhere when running `obj.shine!`, the external service library will send a http request to the internet. We are not worried about what it will return. We are only concerned if the method responsible for sending the request is called.

And that method is `ExternalService.log`. But we are not setting expectations on this log method. Instead the actual method looks like this:

```rb
class ExternalService

  def self.log(hash)
    @service_obj ||= new
    @service_obj.log(hash)
  end

  def initialize
  end

  def log(hash)
    # make API calls!
  end

end
```

Since I don't have any access to the `@service_obj` class-instance var, I had to resort to `instance_variable_set` to set the mock object to it.


Now, this works fine until you have another test case where again you'll have to use this `ExternalService`. That's when this first testcase will affect your second one when you run the whole suite. You'll see this kind of error: **"No more expects available"**

That's because, since you had overridden the original class-instance var with the mock object in one of the testcase, it still remains that way even when you move on to the next case.

This will bite you when you move on to other testcases, in another files, where you are testing some other part of the code, and are not worried about this ExternalService calls (you might've restricted its run based on Rails.env). Those will be affected by this uncleared mock object.

Enter teardown.

In the above testcase, just add a teardown block that resets the class-instance var to nil, and your whole testsuite is good to go!

```rb
  teardown do
    ExternalService.instance_variable_set :@service_obj, nil
  end
```
