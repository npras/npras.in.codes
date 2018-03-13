---
layout: post
title: "Learning and Using Ansible to Deploy Sites"
excerpt: "This post is special. Up its neck, it has the wonderful smell of Configuration Management software."
---

For a long time, I was meaning to never do server configurations manually. Since I knew Chef from a previous [client work](http://npras.in/portfolio/tata/), I was hoping to use it to automate, as first steps, my __\*.npras.in__ family of static sites.

But it wasn't easy to get started with [chef-solo](https://docs.chef.io/chef_solo.html).  There wasn't any proper "Getting Started" guide.

So I was browsing through the alternatives, and quickly found I like Ansible, mainly because it's mostly just yaml.

It is also less complex than Chef. Chef has a bunch of tools/terms under its belt. Recipe, cookbook, librarian, knife, solo, kitchen, attributes etc. And chef needs to be managed as a ruby gem via bundler. I'm a ruby developer yes, but I'm not looking to adore ruby when I'm trying to wrangle servers. I want to get things done.

With Ansible, after a few hours of reading the documentation, I was ready to jump start my explorations. I was able to ping a brand new server that I had just bought in Vultr.

I then decided to ditch my long-serving $5 DigitalOcean server for the awesome $2.5 Vultr server to host my npras.in family of sites.

The final outcome I accomplished after this can be summed up like this:

If all of a sudden, zombies take over the world and destroyed a datacenter where my sites are served, then I'll do these:

* buy a cheap 512MB RAM server from any preferred host (currently preferring Vultr)
* login to the server and `apt-get install python` it. (ansible needs it)
* locally, install ansible. (In macos, it's painfully easy: `brew install ansible`)
* cd into this ansible playbook folder: [https://github.com/npras/ansibles](https://github.com/npras/ansibles)/npras.in.sites.
* run this ansible playbook command to bootstrap the server by installing dependencies (apache, git etc): `ansible-playbook -v -uroot -ihosts.ini server_setup.yml`
* run this ansible playbook command to clone the site repo into the server: `ansible-playbook -v -uroot -ihosts.ini deploy.yml`
* Defeat zombies by living the life of a person with personal websites.

After that, every time I have to publish a new post or edit or remove any content, I'll just have to run the deploy playbook once again to get the changes on server.
