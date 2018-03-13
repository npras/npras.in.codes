---
layout: post
title: "Understanding Turbolinks by Reading Tests"
excerpt: "The javascript library's tests are concise and surprisingly easy to read and understand"
---

The [Turbolinks Readme](https://github.com/turbolinks/turbolinks) documents the library's workings in great details. To fully get it, we might build a rails 5 app and try it out. But there's a faster way to get to know the library and its working - by reading the test cases in the source.

[This section](https://github.com/turbolinks/turbolinks#running-tests) explains how we can first run the whole test suite after cloning the repo locally.

Once cloned and setup, the `bin/blade runner` command runs all test cases (the `QUnit` library is used as the test library).

The test cases are grouped into 3 modules - [navigation_tests.coffee](https://github.com/turbolinks/turbolinks/blob/master/test/src/modules/navigation_tests.coffee), [rendering_tests.coffee](https://github.com/turbolinks/turbolinks/blob/master/test/src/modules/rendering_tests.coffee), [visits_tests.coffee](https://github.com/turbolinks/turbolinks/blob/master/test/src/modules/visit_tests.coffee).

The automated tests are hard to make sense of without trying them manually. For example, [this testcase - "following a same-origin unannotated link"](https://github.com/turbolinks/turbolinks/blob/master/test/src/modules/navigation_tests.coffee#L10) asserts that clicking a TL enabled link,

* triggers a `turbolinks:load` event
* after which the url of the current page is asserted to be the new url `/fixtures/one.html`
* and that, the action that was used to do the `Turbolinks.visit` was 'push', that is, the `history.pushState()` api (and not the `replaceState` or full page load)

Wouldn't it be nice to see it ourselves?

All you have to do is 2 things:

* copy the `turbolinks.js` file from 'dist/' dir to 'test/src/` dir
* cd into the `test/src` dir and serve up all the files using a http server serving command. I use the `serve` command, which is just this alias: `alias serve='echo '\''ruby -run -e httpd . -p 3000'\'' && ruby -run -e httpd . -p 3000'`

And then, followup with the coffee testcases and visit the corresponding files in the fixtures folder.

That's it.

Things I learnt from going through the test cases:

* annotating a link with `data-turbolinks-action=replace` will make it go to a different page, but in the process overwrite the current page's entry in the index. So you can't go back to the page you came from
* `data-turbolinks=false` will make Turbolinks leave the anchor alone. It will just make it do full page load. This the world knows. But an enclosed link within this disabled element will also ignore TL, _unless_ you add a `data-turbolinks=true` to the inner anchor
* `target=\_blank` links always ignore TL and do a full page load
* we know that when a TL enabled link is clicked, the `turbolinks:load` event will be triggered. But before that, `turbolinks:before-render` and `turbolinks:after-render` is fired. (There are many [other events](https://github.com/turbolinks/turbolinks#full-list-of-events) too.)
* all asset elements - link, script etc - are tracked by TL if `data-turbolinks-track=reload` is present. It means that when the assets change in the server, TL will know to do a full page reload instead of just relying on the cached version. That's why the random numbers appended to the application.js and application.css files in rails are very important.
* The `meta` elements are 'accumulated' in the root page of the app. For every TL visit we do that has these meta tags, those elems are gathered and put into root page's head element.
* The javascript present in the head tag of all pages are executed only the first time despite multiple visits. But the javascript present in the body tag of the other pages are executed every time the page is visited. But it seems it is better to rely on the `turbolinks:load` event to do page specific javascripts.
* To programmatically visit a different page in the app, we'll usually do `window.location=/bar.html`. But that makes the browser to do a full page reload not using TL. To programmatically trigger a turbolink enabled url, use `Turbolinks.visit(url)`.


