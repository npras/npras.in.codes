---
layout: post
title: "Monit Chef Recipe To Monitor Rails App In AWS Opsworks"
categories: aws
excerpt: "Monit Chef Recipe That Monitors Rails App Deployed in AWS Opsworks Rails Layer"
---

In a recent client project, I had to write chef recipe that monitors the rails app that was deployed using Opsworks, using monit. Here are the details involved.

## Chef Setup
The Chef recipe file `my_rails_app_cookbook/recipes/my_rails_app_monit.rb`:

```rb
# Installs the monit package
package 'monit'

# Configures the monitrc file based on the corresponding ERB template
template "/etc/monit/conf.d/my_rails_app.monitrc" do
  source "my_rails_app.monitrc.erb"
  owner 'root'
  group 'root'
  mode "0440"
  # Once this resource has been processed (ie, the monitrc file is created), process the next resource (ie, reload monit)
  notifies :run, 'execute[monit_reload]', :immediately
  Chef::Log.info "Creating /etc/monit/conf.d/my_rails_app.monitrc conf file..."
end

# Command resource that executes the "monit reload" command so that monit picks up the new monitrc
execute 'monit_reload' do
  command 'monit reload'
  action :nothing
  only_if { File.exists? '/etc/monit/conf.d/my_rails_app.monitrc' }
  Chef::Log.info "Reloading monit to pick up the latest my_rails_app.monitrc changes..."
end
```

The Chef template file `my_rails_app_cookbook/templates/default/my_rails_app.monitrc.erb`:

```erb
check process apache with pidfile /var/run/apache2/apache2.pid 
  start program = "/etc/init.d/apache2 start" with timeout 60 seconds
  stop program  = "/etc/init.d/apache2 stop"
```

**Note:** This post assumes you are using Apache to run your Rails app in Production. If you are using nginx then you'll have to modify the template above. It shouldn't be hard. You just need to find the pid file's location, and need to know the nginx start and stop command.

## Opsworks Setup
In your "Rails App Layer" in your Opsworks Stack, add this recipe - `my_rails_app_cookbook::my_rails_app_monit` - to the "Configure" phase in the "Custom Chef Recipes" section.

## Verify 
You can now verify that Monit watches the Rails application process and brings it up if it ever goes down. Here are the steps:

* Check if the Rails app process is currently running. You can do this by getting the apache process id (pid) from the pidfile `/var/run/apache2/apache2.pid`, and then 'ps grepping' the pid using this command: `ps -ef | grep <pid>`. You should see the parent process as the apache process, and a few other child processes (passenger and/or apache workers).
* Stop the apache process using either of these 2 commands: `sudo service apache2 stop`, `sudo /etc/init.db/apache2 stop`.
* Verify that the rails app process is not running. "ps grep" again with the same pid you got previously. This time you shouldn't see any results. You can also check by seeing apache's status with `sudo service apache2 status`. It should inform that apache is not running. Finally, you can hit any URL of your rails application and see that there is no response.
* Notice that in a few minutes (or even seconds), the apache process will be up and running. You can check using `sudo service apcache2 status` or `sudo /etc/init.d/apache2 status`. A new pid will be seen in the pidfile and you can ps grep to see its processes. You can also hit any URL of the rails app, and see valid response.

This final step is due to Monit. The monit daemon watches the rails app (actually the apache process), and when it goes down, it start it.
