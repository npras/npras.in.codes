---
layout: page
---

([**portfolio**](/portfolio/) > **parkmobile**)



## Parkmobile


#### __When did you work on this project?__
8 months from April 2015.


#### __What is this project about?__
Parkmobile as a company relies on their main Rails app - the Cape App - that is used by customers to reserve parking slots, pay and collect the passes. The app receives huge number of requests on a daily basis, and more during seasonal events.

This project came to us as they needed a bigger team to handle the workload. I and 2 other devs worked together with their team (from [MalachiArts](http://malachiarts.com/)).

We worked on ongoing user story developments as well as fixing long-standing memory issues.


#### __Who was the client?__
[__Parkmobile USA__](http://us.parkmobile.com/) is the leading Parking Solution provider in USA and UK.


#### __What did _you_ do exactly?__
I worked on new feature requests and fixing bugs. And also was on call to address issues in production app servers.

I remember 2 interesting tasks I owned and delivered:

__Fixing Memory Leak__: The production rails app, which was deployed in 4 servers would have its memory rise high whenever there was more of a specific activity in the site. We’ll get alerted from Nagios once it gets the threshold. It was an established routine that when we get the alerts, we’ll hop on to all the app servers, and restart the apps 1 at a time. This addressed the issue, but we were only treating the symptoms.
I investigated the issue using tools like: rbtrace, memory_profiler, and ObjectSpace, and fixed it. The issue was found in some PDF generating library that was called every time a specific report need to be generated.

__Adhoc script to charge multi-million dollars from Customers__: The ruby script would be run once in a while when a discount has to be applied on a few customer’s purchases. Since we are charging money, I was careful to capture all possible cases where the input might not be as expected, and collect and report relevant errors, and the customer data that caused it. In just a week, I had to write this script. When it was deployed in production, it ran successfully without any incident, while bringing in a few millions in revenue.


#### __Process/Technologies used__
* Rails 4, Ruby 2
* rspec, factorygirl, capybara, cucumber for testing
* capistrano for deployment
* NewRelic for performance monitoring

The company used string Agile methodology. They used Atlassian products extensively (bitbucket, jira, confluence, hipchat etc). JIRA tasks and user stories taken up during the sprint beginning must strictly be delivered. This helped us to really study the problem thoroughly first and analyse them before getting the hands dirty. I attended the scrum meetings once a week, and would update daily status by email.


#### __How did your work impact the company?__
I saved many man-hours and server resources by debugging and addressing the memory-leak issue. One of the proposed workaround after the initial round of failed debugging attempts, was to buy additional server capacity despite having 4 app servers already. But finding and fixing the issue prevented us from going to that workaround.
