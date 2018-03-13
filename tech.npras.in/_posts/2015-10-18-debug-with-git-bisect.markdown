---
layout: post
title: How to Debug with Git Bisect
categories: git
excerpt: Take one step back and then debug
---

Suppose you have an app running in production for the past 5 years. Suddenly your users complain about a bug. You inspect it and find that a particular feature isn't working as expected. You immediately get suspicious as this feature has been in production, working fine, for at least the last 2 years, if your memory serves fine. So it must be due to a change made recently. But you do not know how recently. Git Bisect can tell that, down to the exact commit hash.

Obviously you can't run your test suite to find the broken test. If there was any failing test related to this bug, this wouldn't have sneaked into production code in the first place. You'd have fixed it and then only deployed.

So this means that you now have 2 problems - a production bug, and an untested code somewhere in your app. **<span style="color: #ff0000;">CODE RED: ALERT! DANGER!</span>** You aptly classify it under the aptly named 'PRODISSUES-XXX'.

Your first instinct is to dive into the editor, start reproducing the bug and hunt down looking for it, and to squash it. But something stops you. You pause, and take a step back. You wonder why this bug happened in the first place. After all, yours is a team of smart developers. Sure there can be little carelessness when it comes to testing all the edge cases, but otherwise the changes made by your team should stand to some reason. So instead of pushing a hotfix to this bug, you think it's better to understand the motive behind the changes done. For that you need to see the actual commit that introduced this bug. That way you will be able to pin the problem along with its context and come up with a more appropriate solution, than a mere hotfix.

Git Bisect is your solution then.

Git Bisect does Binary Search. https://www.youtube.com/watch?v=j5uXyPJ0Pew . After starting it, you tell which commit you know has the bad data, usually it's the current commit. And then you tell it which commit you remember doesn't have the issue for sure. That's the good commit. Now git picks a commit that's in the middle of these 2 bad and good commits. The code gets checked out there. You'd have to run the tests to verify if your bug exists in this bug. If it does, you say to git that this too is a bad commit. Now git will pick a commit in between this new bad commit and the original good commit, thereby narrowing down the scope of search, effectively eliminating all the commits between and including the original bad commit and the new bad commit.

Things go on like that, until git has all the info to pinpoint the first commit where the bad code was introduced. You can then see the context in which these changes were made, thereby understanding the bug a lot more clear.

Let's see it by example.

The Calculator class is the place where that bug lurks around. You are sure about it. Here's how it looked in the good old days when you were sure it worked fine.

```rb
class Calculator
  def add(a, b)
    a + b
  end

  def subtract(a, b)
    a - b
  end

  def multiply(a, b)
    a * b
  end

  def divide(a, b)
    a/b
  end

  def modulo(a, b)
    a % b
  end

  def percentage(a, b)
    (a / b) * 100
  end

  def square(a, b)
    a ** b
  end

  def is_equal?(a, b)
    a == b
  end

  def some_really_complex_method(a, b)
    10 + (add(a, b) - subtract(a, b)) * multiply(a, b) / divide(a, b)
  end
end
```

Here's the git log output at this stage.

```
* d61d079 - working calculator class (40 seconds ago) <Prasanna.Natarajan>
```

<span style="line-height: 1.5;">Now imagine 2 years rolled by and a lot of things happened around this code - nothing changing the api though. Here's how the code now looks:</span>

```rb
class Calculator
  # add
  def add(a, b)
    a + b
  end

  # subtract
  def subtract(a, b)
    a - b
  end

  # multiply
  def multiply(a, b)
    a * b
  end

  # divide
  def divide(a, b)
    a/b
  end

  # modulo
  def modulo(a, b)
    a % b
  end

  # percentage
  def percentage(a, b)
    (a / b) * 100
  end

  # square
  def square(a, b)
    a ** b
  end

  # is equal
  def is_equal?(a, b)
    a == b
  end

  # complex method
  def some_really_complex_method(a, b)
    10 + (add(a, b) - subtract(a, b)) * multiply(a, b) / divide(a, b)
  end
end
```

  <span style="line-height: 1.5;">I've just added comments. No changes to the actual code. Here's how the git log now looks like:</span>

```
* 32a7457 - (HEAD, master) even more comments (1 second ago) <Prasanna.Natarajan>
* 478c0a2 - more comments (44 seconds ago) <Prasanna.Natarajan>
* 24eb9f0 - comments (78 seconds ago) <Prasanna.Natarajan>
* d61d079 - working calculator class (4 minutes ago) <Prasanna.Natarajan>
```

  Bear with me a bit more. Another 2 years roll by. The code now looks like this:

```rb
# Copyright notice:
#
# Calculator class. To calculate!
#
# TODO: Add sin, cos, tan, log, base, square root formula calculation.
#
# add class comment
class Calculator
  # add
  def add(a, b)
    a + b
  end

  # subtract
  def subtract(a, b)
    a - b
  end

  # multiply
  def multiply(a, b)
    a * b
  end

  # divide
  def divide(a, b)
    a/b
  end

  # modulo
  def modulo(a, b)
    a % b
  end

  # percentage
  def percentage(a, b)
    (a / b) * 100
  end

  # square
  def square(a, b)
    a ** b
  end

  # is equal
  def is_equal?(a, b)
    a == b
  end

  # identity
  def identity(a)
    a
  end

  # complex method. It's a really complex method
  def some_really_complex_method(a, b)
    11 + (add(a, b) - subtract(a, b)) * multiply(a, b) / divide(a, b)
  end
end
```

  <span style="line-height: 1.5;">The log now looks like this:</span>

```
* d7c9adf - (HEAD, master) copyright notice added (1 second ago) <Prasanna.Natarajan>
* 309cf66 - more TODO's (48 seconds ago) <Prasanna.Natarajan>
* 10878d0 - identity method added (2 minutes ago) <Prasanna.Natarajan>
* 81b49af - irrelevant meaningless comment (3 minutes ago) <Prasanna.Natarajan>
* abad852 - add todo to write code for sin, cos, tan formula (4 minutes ago) <Prasanna.Natarajan>
* 07f44e4 - class description comment (5 minutes ago) <Prasanna.Natarajan>
* fc05c7e - class comment (6 minutes ago) <Prasanna.Natarajan>
* 32a7457 - even more comments (10 minutes ago) <Prasanna.Natarajan>
* 478c0a2 - more comments (11 minutes ago) <Prasanna.Natarajan>
* 24eb9f0 - comments (11 minutes ago) <Prasanna.Natarajan>
* d61d079 - working calculator class (14 minutes ago) <Prasanna.Natarajan>
```

  I've just added the copyright notice comment, and more TODOs. **But notice that the bug is introduced in this set of changes.** The body of the method `some_really_complex_method` has changed from:

```rb
10 + (add(a, b) - subtract(a, b)) * multiply(a, b) / divide(a, b)
```

to

```rb
11 + (add(a, b) - subtract(a, b)) * multiply(a, b) / divide(a, b)
```

10 got changed to 11\. Stupid bug, I know. Nevertheless a bug. Note also the culprit commit `fc05c7e`. It's the commit that introduced this bug subtly. (This is the commit we are going to find through git bisect after a user reports a bug.)

### A Bug Day

Time has passed. It is now the present day. You wake up to a red alert mail saying that a bug is present in the production. Since you want the commit this bug was introduced, you decide to use bisect. But before starting, you notice there's no test coverage around the suspect class `Calculator`! A Cardinal sin, yes, but this is no time to get medieval on anybody. We have a production problem to deal with first, Houston. You quickly write a minimal test case covering the expected behavior of the Calculator class. This is in a separate file named `calculator_spec.rb`. Here's how it looks:

```rb
require 'minitest/autorun'
require './calculator'

describe Calculator do
  before do
    @calc = Calculator.new
  end

  it 'should return the result of a complex calculation from #some_really_complex_method' do
    _(@calc.some_really_complex_method(100, 50)).must_equal 2_50_010
  end

end
```

  This test fails in the current commit. Here's the test result:  

```
1) Failure:
Calculator#test_0001_should return the result of a complex calculation from #some_really_complex_method [calculator_spec.rb:10]:
Expected: 250010
Actual: 250011
```

That's obvious. We know the bug is present in this commit. What we don't know is the commit that introduced this bug. **The Show begins here.** Start git bisect first with this terminal command:

```
$ git bisect start
```

  You now need to provide it a range where it can scope its searches to. You have to tell it a bad commit where you know the bug exists, and also a good commit, somewhere from the past, where you are sure that the bug didn't exist. Git will then use the commits between these 2 commits as search space to find the problem causing commit. Tell git that the current commit is the bad commit:

```
$ git bisect bad
```

  Tell it a good commit:

```
$ git bisect good d61d079
```

  I'm giving as good commit the commit where I'm positive everything works fine, the very first commit. The output now is this:

```
Bisecting: 4 revisions left to test after this (roughly 2 steps)
[07f44e4f02457a405c0dd19406a705a5bdb2a7ce] class description comment
```

What it means? Git says that in about 2 steps we'll be able to spot the bad commit. It has also checked out the code at a commit in between the bad and good commit points we had mentioned above. We now have to run our test to see if it passes.

```
1) Failure:
Calculator#test_0001_should return the result of a complex calculation from #some_really_complex_method [calculator_spec.rb:10]:
Expected: 250010
Actual: 250011
```

Looks like we still have the bug. We now tell git that this commit too is a bad commit.

```
$ git bisect bad
```

  Now git shows this output:

```
Bisecting: 2 revisions left to test after this (roughly 1 step)
[478c0a27988a371d621b7b7fccc8df4c1030474c] more comments
```

Git has now checked out a commit in between the previous mid-commit (which we labeled as bad) and the original good commit. Let's run the test now.

```
# Running:

.

Finished in 0.001230s, 813.0333 runs/s, 813.0333 assertions/s.
```

  The specs pass! It means that the problem causing code was introduced after the currently checked out commit. Let's tell git that this is a good commit.

<pre class="lang:default decode:true">$ git bisect good</pre>

<pre class="lang:default decode:true "># output
Bisecting: 0 revisions left to test after this (roughly 1 step)
[fc05c7ef692ef63e351a0513dd5ee546577bd75b] class comment
</pre>

  <span style="line-height: 1.5;">Git has checked out a commit above the previously checked out commit. Git is still not sure if this is it. It needs more assurance from us. Let's run the test to see if we can give it.</span> The test failed again! Tell git that this one is a bad commit:

<pre class="lang:default decode:true">$ git bisect bad
Bisecting: 0 revisions left to test after this (roughly 0 steps)
[32a7457b6ab3053da16a1cab258a3092b55c65fd] even more comments</pre>

  Run tests again. Test passes. Tell git that this is a good guy. See the magic happen now.

<pre class="lang:default decode:true" title="Magic output!">$ git bisect good
fc05c7ef692ef63e351a0513dd5ee546577bd75b is the first bad commit
commit fc05c7ef692ef63e351a0513dd5ee546577bd75b
Author: Prasanna.Natarajan <prasanna@npras.in>
Date: Sun Oct 18 07:20:17 2015 +0530

class comment

:100644 100644 29aae0953045ad92b5c43b68d4c4889c64e22bfc 6b8b59b664318728fc9d4e4a9e33657c3af1ac72 M calculator.rb</pre>

  That's `git bisect` coming up with a conclusion. It has found the problem commit to be the one labeled `class comment`. Sure enough if we see the changes introduced by this commit, we can see the line that caused the bug in all future versions of the code:

<pre class="lang:diff decode:true" title="git show output">$ git show fc05c7ef692ef63e351a0513dd5ee546577bd75b
commit fc05c7ef692ef63e351a0513dd5ee546577bd75b
Author: Prasanna.Natarajan <prasanna@npras.in>
Date: Sun Oct 18 07:20:17 2015 +0530

class comment

diff --git a/calculator.rb b/calculator.rb
index 29aae09..6b8b59b 100644
--- a/calculator.rb
+++ b/calculator.rb
@@ -1,3 +1,4 @@
+# add class comment
class Calculator
  # add
  def add(a, b)
@@ -41,6 +42,6 @@ class Calculator

# complex method
def some_really_complex_method(a, b)
-   10 + (add(a, b) - subtract(a, b)) * multiply(a, b) / divide(a, b)
+   11 + (add(a, b) - subtract(a, b)) * multiply(a, b) / divide(a, b)
  end
end</pre>

  Yuck! Instead of just adding a comment, I've accidentally changed a value in a method, that too in a `complex` method! Now you can reset to your normal workflow by:

<pre class="lang:default decode:true ">git bisect reset</pre>

  and fix the bug and live a good life.

### Conclusion

This contrived example might not showcase bisect's full power, but when used in a real project spanning thousands of files and commits, because of binary search, git bisect will help us to arrive at the problem much quicker that you'd be able to find manually. Note: Git Bisect can be used to debug programs in any language, not just ruby.
