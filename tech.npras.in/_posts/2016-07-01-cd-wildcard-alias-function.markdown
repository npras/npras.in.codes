---
layout: post
title: "Bash cd wildcard alias function"
categories: linux
excerpt: Don't wait for auto-completion to change directory!
---

Quickly add this to your `.bashrc` or wherever you have your favorites bash aliases setup:

```shell
cdp() {
  cd *$1*
}
```

You are welcome!


### For the clueless

Imagine are navigating around folders in a terminal.

In your current directory, you have two dirs named:

* test_models_1_database
* test_models_2_business

Let's suppose you want to navigate to the second one.

Normally this is what you'll do:

* press 't' and then 'tab'
* bash completion would help you till `test_models_`
* you'll then press '2' to get the full desired path autocompleted.

With the above function though, all you have to do is:

```shell
cdp bus
```

You are now already into that folder!
