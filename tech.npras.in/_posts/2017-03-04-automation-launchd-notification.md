---
layout: post
title: "MacOS Automation: Learning launchd and Notifications"
excerpt: "My great Mac has its own API, and it's a shame I haven't learnt to use it until now."
---

This is the story of how I wanted to do 1 thing, and learnt 2 things to get that 1 thing:

* how to create new [notifications](https://developer.apple.com/notifications/)
* how to use the cron-like utility - [launchd](https://developer.apple.com/library/content/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html) - to run specific scripts.
* how to have done that 1 thing without spending time to learn these 2 things.

I use the great [SelfControl App](http://selfcontrolapp.com/) to block distracting sites whenever ~~I'm working~~ I'm using my laptop. (__Unnecessary-to-this-topic-but-necessary-to-life Aside:__ There's an easy way to bypass this app's restrictions. In the 'Date & Time' setting, just set the time manually to 1 day in future. This will confuse SelfControl and allow you to visit the restricted sites easily! You can even reset the time again and SelfControl won't know. But I've grown to pause and reflect before trying to game the system. The app works effectively _that way_.)

The days that I start with SelfControl turned on are my most productive days. The days I forget SelfControl are days when the the _Big Corps_ have succeeded in tricking me to sell my attention to them.

So, every time I start my laptop, I want to be reminded, by a system notification, that I need to start the app.

I also want to "brew doctor" and "brew update" my homebrew software daily so that I'm always having the correct version.

To do so, we'll need to write the `display notification` applescript (which is actually javascript).

```js
// saved in: /Users/prasanna/bin/launchd_scripts/notification_startup.js

var app = Application.currentApplication();

app.includeStandardAdditions = true;

app.displayNotification("Run brewup and start SelfControl", {
    withTitle: "Startup Reminders",
    soundName: "Frog"
});
```

The message that I want in the notification is: "Run brewup and start SelfControl".

You can see it working right away by running this file using the `osascript` command that's used to run applescript scripts.

```shell
osascript /Users/prasanna/bin/launchd_scripts/notification_startup.js
```

Now, all we have to do is just schedule it using launchd. To do that we need to create a `plist` file in `~/Library/LaunchAgents`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">

<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.example.startup</string>

    <key>ProgramArguments</key>
    <array>
      <string>osascript</string>
      <string>/Users/prasanna/bin/launchd_scripts/notification_startup.js</string>
    </array>

    <key>RunAtLoad</key>
    <true/>

    <key>KeepAlive</key>
    <dict>
      <key>SuccessfulExit</key>
      <false/>
    </dict>

  </dict>
</plist>
```

Now, on the next login, the `launchctl` program will load all the plist files in this dir and execute them as per their config.

This file is configured to run at startup. So, after login, this notification will pop up, and I'll be __PRODUCTIVE!!__.

(Just after learning and implementing this, I realised that we could easily have any app start at start up, using the 'System Preferences' > 'User & Accounts' > 'Login Items' menu. I wept.)
