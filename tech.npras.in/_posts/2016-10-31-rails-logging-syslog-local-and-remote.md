---
layout: post
title: "Rails Logging: rsyslog, /var/log/, logrotate and a Chef Recipe to bind these together"
categories: rails
excerpt: Teach Rails to log at /var/log/ via rsyslog, rotate it via logrotate, and automate both via Chef
---

I worked on a project where a client wanted the production app's log to be written to a file in the `/var/log/` directory. Another requirement was to disable Rails from doing the actual logging, and use the ubuntu's central logging service `rsyslog` to do that. The client also wanted the log files to be rotated automatically depending on size.

Here's how it was implemented:

## rsyslog and /var/log
The default rails logger is set in the rails app's config variable `config.logger`. This has to be changed to make it point a different logger that sends the logs to the central syslog service.

Ruby has the [Syslog](https://ruby-doc.org/stdlib-2.3.1/libdoc/syslog/rdoc/Syslog.html) Standard Library that does the heavy lifting for you. You can set the Rails app's `config.logger` object using the syslog object created from this library, like so:

```ruby
# In your environment config file such as production.rb
require 'syslog/logger'
config.logger = ActiveSupport::TaggedLogging.new(Syslog::Logger.new('my_rails_app'))
```

Note: The use of `TaggedLogging` is not mandatory here. You can simply use `ActiveSupport::Logger`. But I've come to take advantage of tagged logging in recent times as it provides context to the log parsers.

The 'program_name' argument you give to `Syslog::Logger` - in this case `my_rails_app` - is the name rsyslog will use to distinguish the logs from this rails app with other logs generated by other processes.

With just this setup, and restarting the application, you can see that the app now logs to the `/var/log/syslog` file, instead of the usual location.

Now, we can configure rsyslog to make it send this app's logs to a specific file `/var/log/my_rails_app.log`. To do this, you have to add a custom rsyslog config file: `/etc/rsyslog.d/my_rails_app.conf`. Its contents will be:

```conf
if $programname == 'my_rails_app' then /var/log/my_rails_app.log
```

Now, restart rsyslog using either `sudo service rsyslog restart` or `sudo service rsyslogd restart`.

Now the rails app's logs can also be seen in the file /var/log/my_rails_app.log. Note that you don't have to create or 'touch' this file. It will be created automatically by rsyslog.

---

## Logrotate
Setting up logrotate to compress and rotate the log files is pretty easy. Just add this config file at `/etc/logrotate.d/my_rails_app`. Its contents will be:

```conf
/var/log/my_rails_app.log {
       	rotate 7
       	size 10M
        missingok    
        compress     
        delaycompress
        notifempty   
        copytruncate
}
```

There's no need to restart any service here, as logrotate is run once a day based on a cronjob. So when the time arrives, this new config file will be picked up along with other config files.

---

## Chef Recipes

If you are deploying your rails app with these setup, and you are automating the deployment via AWS's Opsworks or just vanilla Chef, the following chef recipe, along with the necessary templates (all shared below), will get the job done.


The main logging recipe file in your cookbook `recipes/my_rails_app_logging.rb`. Its contents are:

```rb
template "/etc/rsyslog.d/my_rails_app.conf" do
  source "my_rails_app.rsyslog_conf.erb"
  owner 'root'
  group 'root'
  mode '0644'
  notifies :run, 'execute[rsyslog_restart]', :immediately
  Chef::Log.info "Creating /etc/rsyslog.d/my_rails_app.conf rsyslog conf file..."
end

template "/etc/logrotate.d/my_rails_app" do
  source "my_rails_app.logrotate_conf.erb"
  owner 'root'
  group 'root'
  mode '0644'
  Chef::Log.info "Creating /etc/logrotate.d/my_rails_app logrotate conf file..."
end

execute 'rsyslog_restart' do
  command 'service rsyslog restart'
  action :nothing
  only_if { File.exists? '/etc/rsyslog.conf' }
  Chef::Log.info "Restarting rsyslog to take the new my_rails_app.conf into account..."
end
```

The recipe does 3 things:

* creates the rsyslog conf file for the application
* creates the logrotate conf file for the application
* restarts the rsyslog service so that the rsyslog conf file takes effect

The templates are:

```rb
# templates/default/my_rails_app.rsyslog_conf.erb
if $programname == 'my_rails_app' then /var/log/my_rails_app.log
```

```rb
# templates/default/my_rails_app.logrotate_conf.erb
/var/log/my_rails_app.log {
       	rotate 7
       	size 10M
        missingok    
        compress     
        delaycompress
        notifempty   
        copytruncate
}
```