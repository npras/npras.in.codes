---
layout: post
title: "Directory Structure and File Requiring of A Ruby Commandline Program"
categories: ruby
excerpt: "Always get confused with require and require_relative? I know!"
---

It is not that easily believable. But yes, occasionally, a Rails developer might have to write a simple Ruby commandline script. It is a fact, and all facts should be respected.

A Ruby commandline script is not a gem. But its folder structure can be similar to that of a gem.

For the commandline functionality, you can use any of the many available utility gems. Popular choices are `rake`, `thor`, `optparse`. I prefer `thor` as its API doesn't deviate much from the standard PORO.

Here's the bare minimum directory structure of such a program.

```
$ tree
.
├── Gemfile
├── Gemfile.lock
├── bin
│   └── the_command
├── lib
│   ├── commandline_ruby
│   │   ├── cli.rb
│   │   └── main.rb
│   └── commandline_ruby.rb
└── test

3 directories, 7 files
```

The main file `lib/commandline_ruby.rb` can be used for 2 purposes:

* requiring all the relevant files that are present in other dirs in the lib folder
* Defining the main module of this library, and defining common utility classes and functions in this module. This is how it can look:

```rb
module CommandlineRuby
  extend self

  # utility functions
  def logger
  end
end

require_relative 'commandline_ruby/main'
require_relative 'commandline_ruby/cli'
```

It's crucial that the files are loaded using `require_relative`. Only then the executable file `bin/ can be invoked from anywhere.

Next is the `lib/commandline_ruby/cli.rb` file. This is the file where the classes and code related to your chosen commandline library will go. In my case, it would be place where the Thor main class is defined. It can look like this:

```rb
require 'thor'

module CommandlineRuby
  class CLI < Thor

    desc "process", "do something useful!"
    def process
      Main.new.process!
    end

  end
end
```

I like to extract out the actual business logic out of these utility classes and put them in their own classes. That's why the `CLI` class seems to be doing nothing much. It delegates the meat of the task to the `Main` class.

`lib/commandline_ruby/main.rb` file is where this script's main work is being done. It can look like this:

```rb
module CommandlineRuby
  class Main

    include Module1
    include Module2

    def process!
      # The MEAT!
    end

  end
end

```

From this class, you can delegate the work to many other classes and modules, all of which will be already loaded in the `lib/commandline_ruby.rb file.

The last, but the most important file is the actual executable file. This will expose this script's functionality as a command that you can run from the terminal. It can look like this:

```rb
#!/usr/bin/env ruby
require_relative "../lib/commandline_ruby"

CommandlineRuby::CLI.start(ARGV)
```

Since we follow a convention (which is industry standard) of requiring all dependencies in 1 file before the actual work ever starts, all of our requires would be in the `lib/commandline_ruby.rb` file. (Actually, they are 'require_relative's, not 'require's.)

This executable file in the bin directory just requires the main file, and just calls the main method of the commandline library's code. In my case here, it would be the Thor class's `start` method, passing in the arguments we got from the commandline.

## Outro
This is a simple way to structure your non-rails, commandline programs. I've used them in many client projects. They run perfectly even in Production environment. Let me know if I can improve it by any means.
