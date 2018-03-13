---
layout: post
title: "Securing a blank server with Ansible and Let's Encrypt"
excerpt: "More ansible and more security paranoia!"
---

Today I updated the ansible playbook that sets up and deploys the \*.npras.in sites, to enforce a bit of security and also made my sites serve over https using [Let's Encrypt](https://letsencrypt.org/).

I accidentally came upon the fact that many hacker botnets are _constantly_ trying to hack into my server. My ansible playbook wasn't able to ssh into the server, while I could ssh manually. So the internet said to look at the `/var/log/auth.log` file. It logs every ssh attempt. Looking at it, I was horrified to find numerous attempts to ssh into my server from ips across the world. They were trying dictionary attacks it seems. With just the knowledge of my ip address (they could easily find the ip address _range_ of all AWS servers for example), the default ssh port and "root" as the user, the botnets can be triggered to attempt to gain access to almost all servers in the world.

Of course, all of those attempts failed (I think!) as they didn't have the password or the private ssh key that I used. But the sheer volume of the attempt increased the bandwidth usage a lot. The network graph in the Vultr dashboard showed about 50 MB of usage since the day I bought the server. Even though I don't have Google Analytics, I was _positive_ that my websites didn't attract that much traffic. It was the ssh attempts. While Vultr gives 500GB bandwidth per month, which could easily withstand the attacks without exceeding the limits, I wanted less holes in the ozone layer.

So the fix I 'ansibled' was:

* to change ssh default port from 22 to something obscure
* to install `ufw` firewall and disable all traffic as the first policy, and enable only on need basis. (ssh, http and https)
* disallow password based authentication. Only key-based authentication is allowed.


While I'm not a security expert, this _did_ considerably reduce the botnet attempts, thereby saving precious bandwidth.

I'm planning to read upon some best practices and trying to implement them using ansible.

While going through the wonderful blog of [Julia Evans](https://jvns.ca/), I came across "Let's Encrypt" and how it's a quick way to add ssl to your sites.

I looked up the [ansible module for it](https://docs.ansible.com/ansible/letsencrypt_module.html), but I couldn't understand it's usage. I think we have to generate the csr file manually and then also do the challenge fulfillment step. I declined the challenge.

I went through the docs and found that the [manual installation](https://certbot.eff.org/#ubuntutyakkety-apache) steps were painfully simple and quick.

So I did that, and 💣 just like that, I had my sites served over https.

More importantly, I learnt today to use emojis using the simple but elegant [Rocket app](http://matthewpalmer.net/rocket/). You should now be able to tell a pale poop from a dark poop. Here is the dark one: 💩.
