---
layout: post
title: "Programmatically Accessing Sprockets Assets"
excerpt: "Adding preprocessed styles/js inline"
---

Suppose you have a css or js asset file, and you want to include it in your layout file *inline*, and not via `<script src="...">` or `<link href="...">`.

You could do so by accessing the asset file's contents as a ruby string via the `Rails.application.assets.find_asset` method.

Here's how to load inline a script file that's in your asset pipeline path:

```html
<script>
<%= Rails.application.assets.find_asset("my-custom-work.min.js").to_s.html_safe %>
</script>
```

And here's how a css file can be included within a layout. First add a helper method, and then use it in the layout.

```rb
def inline_styles(stylesheet)
	Rails.application.assets.find_asset("#{stylesheet}.css").to_s.html_safe
end
```

```html
<style>
  <%=raw inline_styles(@stylesheet) %>
</style>
```

Note that since we might've added these assets to the precompiled list, we could edit those asset files using our favorite processing language - scss, coffeescript etc.
