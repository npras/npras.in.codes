---
layout: post
title: "AWS Opsworks Custom Layer: Chef Recipes to Deploy Custom Ruby Scripts"
categories: aws
excerpt: "Learn how to write Chef Recipes for Non-Rails Ruby Apps"
---

Recently in a client project, I had to configure 2 apps using AWS Opsworks.

One was a standard Rails app. Since Opsworks comes with a 'Rails Layer', I used that. And using Chef 11.10 meant, Opsworks gave me an easy heads-up for things like Apache and Passenger, deployment recipes, Bundler, Rubygems etc

The other app was a Ruby script that is supposed to run as a Cron job. Since I can't use the 'Rails Layer', I had to use the 'Custom Layer' and write all the setup and deployment Chef Recipes.

The first piece of the problem is the deployment recipe. Chef has a [deploy resource](https://docs.chef.io/resource_deploy.html). It's a complete package to do Capistrano-style deployment in the server. But since it has a lot of configuration to be made, I decided to use the Deployment recipe(s) used by an Opsworks 'Rails Layer'. Exploring one such stack, I found that the 'Deploy' event runs 2 built-in recipes: [deploy::default](https://github.com/aws/opsworks-cookbooks/blob/release-chef-11.10/deploy/recipes/default.rb) and [deploy::rails](https://github.com/aws/opsworks-cookbooks/blob/release-chef-11.10/deploy/recipes/rails.rb).

For this custom ruby script, we won't need the `deploy::default` recipe. It would just create a deploy user (I think). It might be useful later on, but for now, we'll just focus on getting a deployment succeed. For that, we need to tweak the `deploy::rails` recipe to remove the rails specific part and keep the generic parts.

Here's the final deploy recipe we arrived at:

```rb
include_recipe 'deploy'

node[:deploy].each do |application, deploy|
	opsworks_deploy_dir do
    user node[:opsworks][:deploy_user][:user]
    group node[:opsworks][:deploy_user][:group]
    path deploy[:deploy_to]
  end

  opsworks_deploy do
    deploy_data deploy
    app application
  end

  gem_package 'bundler'

  execute 'bundle install' do
    cwd "#{deploy[:current_path]}"
    command "bundle install"
  end

end
```

Note that we are running bundler on every deploy. It's because we used it to manage the scripts dependencies. You can ignore it if not needed.

This deployment recipe works well. On each Opsworks deploy, we've made this recipe to run as a custom 'deploy' event recipe. It deploys the app in the same way Capistrano would.


## The Cron Recipe
Since this ruby script is to be run from Cron, every X minutes, we need to update the crontab in the server too. Here's a custom recipe for that:

```rb
cron_mins = node[:deploy]['reportscript'][:environment_variables][:cron_mins]

unless cron_mins
	cron_mins = 15
end

cron_env = {"PATH" => "/usr/local/bin"}

cron "generate_report" do
  environment cron_env
  minute "*/#{cron_mins}"
  command "cd #{node[:deploy]['reportscript'][:current_path]} && ./bin/report_script process >> #{node[:deploy]['reportscript'][:current_path]}/log/cron.log 2>> #{node[:deploy]['reportscript'][:current_path]}/log/error.log"
end
```

Note that this recipe can take arguments in the form of **Custom JSON**, that would be exposed by Chef within the running process as `environment_variables`. Using the variable `cron_mins`, you can change the cron interval within which the app would run.

Also, this recipe will not be run as part of the deploy event. It will be run manually via the Opsworks' "Run Command" interface. You'll need to provide the arguments in custom json.
