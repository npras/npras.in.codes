---
layout: post
title: Uploading database.yml to Server using Capistrano
categories: code
excerpt: this is what the internet was missing
---

There are many solutions available to the problem of uploading git-ignored, sensitive data to the server. Files like `database.yml`, `secrets.yml`, `.env.production` and other files where api keys are stored need to be uploaded into the server safely. A simple and safe solution is to use Capistrano's built-in `upload!` method. An example usage can be seen in [Capistrano's examples page](https://github.com/capistrano/sshkit/blob/master/EXAMPLES.md#upload-a-file-from-disk). To elaborate on the example to upload all the above mentioned files to the server, create a rake task file named as `upload_config_files.rake` in your app's directory `lib/capistrano/tasks` with this content:

<pre class="lang:ruby decode:true ">namespace :setup do

  desc "upload protected config files to shared_path"
  task :upload_config do
    on roles(:all) do |host|
      upload! 'config/database.yml', "#{shared_path}/config/database.yml"
      upload! '.env.production', "#{shared_path}/.env.production"
      upload! 'config/secrets.yml', "#{shared_path}/config/secrets.yml"
    end
  end

end
</pre>

Now when you run the `cap -T` command from the terminal, you'll see this new task added to the list of available cap tasks.

<pre class="lang:sh decode:true ">$ cap -T | grep upload
cap setup:upload_config      # upload protected config files to shared_path</pre>

Since this is not hooked into any existing deploy tasks, it will not be run automatically each time you deploy. You have to make sure you run it before your first deploy (or as and when needed). You should also have created the directories manually mentioned in the file path above. In my experience, capistrano doesn't do it by itself. To keep them part of the shared_path directories, add this to your `config/deploy.rb` file:

<pre class="lang:ruby decode:true">set :linked_files, fetch(:linked_files, []).push('config/database.yml', '.env.production', 'config/secrets.yml')
</pre>
