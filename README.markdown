# jekyll code for my sites

* http://npras.in - my portfolio site
* http://tech.npras.in - my tech blog
* http://diary.npras.in - my book notes and ramblings

## Deployment

All these jekyll codes generate their sites to a different repo: https://github.com/npras/npras.in.sites.

Deployment happens through git's `post-update` hook.

The hook is present in the `production` remote (bare git repo) present in the server.

Here's the hook code, present in `/home/deployer/npras.in.sites.git/hooks/post-update` file:

```shell
#!/bin/sh

#
# An example hook script to prepare a packed repository for use over
# dumb transports.
#
# To enable this hook, rename this file to "post-update".

GIT_WORK_TREE=/var/www/html/npras.in.sites git checkout -f
```

The file is an executable.

The dir `/var/www/html/npras.in.sites` is owned by `deployer`, and group `www-data` has access to it.

`deployer` user is added to `www-data` group.

Every time a site is updated, I push to 2 remotes:

```shell
$ git push # origin master
$ git push prod # master
```

The 2nd remote - `prod` - points to my server's bare git repo:

```shell
$ git remote -v
origin	git@github.com:npras/npras.in.sites.git (fetch)
origin	git@github.com:npras/npras.in.sites.git (push)
prod	prasblog:/home/deployer/npras.in.sites.git (fetch)
prod	prasblog:/home/deployer/npras.in.sites.git (push)
```

So a push to this remote triggers the `post-update` hook defined above in the server.

The hook will force-checkout the working tree present at the (another) local repo present in the server at `/var/www/html/npras.in.sites`.
`GIT_WORK_TREE` dynamically sets the working tree (dir) path for the command `git checkout -f`

The deploy is done!

### How to create a bare git repo in the server
#### at server
* cd
* mkdir npras.in.sites.git && cd into it
* git init --bare
* create the git hook as mentioned above
#### at local
* git remote -v
* g remote add prod prasblog:/home/deployer/npras.in.sites.git
* git remote -v
origin	git@github.com:npras/npras.in.sites.git (fetch)
origin	git@github.com:npras/npras.in.sites.git (push)
prod	prasblog:/home/deployer/npras.in.sites.git (fetch)
prod	prasblog:/home/deployer/npras.in.sites.git (push)
* git push prod master

### Apache Virtualhost .conf files for these sites


* /etc/apache2/sites-available/diary.npras.in.conf

```
<VirtualHost *:80>
  ServerName diary.npras.in
  ServerAlias diary.npras.in

  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/html/npras.in.sites/diary.npras.in
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined

  <Directory /var/www/html/diary>
    AllowOverride All
  </Directory>

</VirtualHost>
```


* /etc/apache2/sites-available/tech.npras.in.conf

```
<VirtualHost *:80>
  ServerName tech.npras.in
  ServerAlias tech.npras.in

  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/html/npras.in.sites/tech.npras.in
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined

  <Directory /var/www/html/tech>
    AllowOverride All
  </Directory>

</VirtualHost>
```


* /etc/apache2/sites-available/books.npras.in.conf

```
<VirtualHost *:80>
  ServerName books.npras.in
  ServerAlias books.npras.in

  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/html/npras.in.sites/books.npras.in
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined

  <Directory /var/www/html/books>
    AllowOverride All
  </Directory>

</VirtualHost>
```


* /etc/apache2/sites-available/npras.in.conf

```
<VirtualHost *:80>
	ServerName npras.in
	ServerAlias www.npras.in

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html/npras.in.sites/npras.in

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```
