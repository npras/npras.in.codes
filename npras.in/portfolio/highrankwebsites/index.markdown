---
layout: page
---

([**portfolio**](/portfolio/) > **highrankwebsites**)



## HighRankWebsites.com


#### __When did you work on this project?__
5 months in 2015.


#### __What is this project about?__
The client had a set of tools like RankTracker, LinkProspector and KeywordAnalysisTool that provide additional value to his existing [HighRankWebsites](https://highrankwebsites.com) customers.

[__Rank Tracker__](https://tracking.highrankwebsites.com/)

The site tracks the search engine ranks for various long tail terms of the customer's website.

The home page lists all the customer websites and their overall stats like average rank, whether it was increased/decreased in the past few days or so, the SEO person in assigned to the website. Here's the homepage snapshot:

<img class="portfolio" src="/assets/images/portfolio_rt_homepage.png" alt="RankTracker Homepage">

On clicking individual websites, we get a nice overview of the individual rankings for the keywords the customer wanted to improve the ranking for in google. The data is given as both a graph and a table. If the customer has provided the google analytics id for his site, then that data is also pulled and shown as graphs and percentages.

<img class="portfolio" src="/assets/images/portfolio_rt_keywords.png" alt="RankTracker Keywords">

Also, there's a profile page for each website that details other stats like these. Metrics like domain authority/inbound links are pulled from SEOMoz api. The social metrics are pulled from sharedcount site.

<img class="portfolio" src="/assets/images/portfolio_rt_website_profile.png" alt="RankTracker Website Profile">

The SEO person (a team member of Mike's company) assigned to this site is responsible for bringing the customer's site on top of google for various related keywords. They do that by building links. They add the customer's business details along with his website to other websites that have a high page rank - like a business directory. The more high PR link we get for the customer, the higher his site will show up in google. Here's a linktracking page for a customer: (the same apis are used to get the metrics for these links)

There's also an events page for each customer/website:

<img class="portfolio" src="/assets/images/portfolio_rt_events.png" alt="RankTracker Events">

There's also a to-do page and a notes page for each customer. But they are tiny features and are seldom used I guess.

And then, there's the admin level nav bar enabled only for admin level users. An admin can view all the events, all the link trackings, edit/delete/create new accounts etc. Here's the admin level link tracking page:

<img class="portfolio" src="/assets/images/portfolio_rt_all_linktracin.png" alt="RankTracker Link Tracing">


[__Link Prospector__](https://qlp.highrankwebsites.com/campaigns)

Note: Both these tools work on a common database and a common goal.

This link prospector tool is used to create specific campaigns for a customer targeted around a particular page in his website or a new event happening in his site - eg: book lauch.

The home page lists all the campaigns created so far for all of the customers(websites):

<img class="portfolio" src="/assets/images/portfolio_qlp_campaigns.png" alt="">

A campaign can be created by this form after selecting the customer for whom it is needed:

<img class="portfolio" src="/assets/images/portfolio_qlp_newcampaign.png" alt="">

The main goal of a campaign is to trigger a google search to find high PR sites that also has a contact detail of the decision-maker in that site. These contacts are called prospects. A custom email, relevant to the customer and relevant to the just-found site is sent to the prospect. This looks like a way of promoting our customer's site/product. A campaign can have many search terms. Here's one search that was able to find some contacts(prospects):

<img class="portfolio" src="/assets/images/portfolio_qlp_prospects.png" alt="">

And here's the email sent to the prospect:

<img class="portfolio" src="/assets/images/portfolio_qlp_sentemail.png" alt="">

Here's the detail of a prospect page:

<img class="portfolio" src="/assets/images/portfolio_qlp_prospect_page.png" alt="">

The emails sent are tracked for clicks/opens/replies via the mailgun api. Here's an admin level graph showing the email stats:

<img class="portfolio" src="/assets/images/portfolio_qlp_teamreport.png" alt="">

The 'Media' section is nothing but the prospect links that are manually classified as media. Eg: cnn, times, huffington post etc.

Each contact/prospect found have a separate contacts page with their twitter stats, emails sent to them etc:

<img class="portfolio" src="/assets/images/portfolio_qlp_contact.png" alt="">



#### __Who was the client?__
[__HighRankWebsites__](http://www.highrankwebsites.com/). An SEO firm that does many auxillary services like Ad Management, PPC tracking, Social media sharing and keyword analysis.

For this project, I gathered requirements and reported the ongoing process directly to the CEO Mike Perez.


#### __What did _you_ do exactly?__
The app was already developed and deployed when it came to me. I was in charge of maintaining, new feature development, bug fixes and server maintenance.


#### __Process/Technologies used__
* [Cuba framework](http://cuba.is) A minimal webapp framework very much unlike Rails!
* Postgresql for database and [Sequel gem]() for interacting with it.
* Ruby 2.1

Sinatra and the Cuba framework was used to build the 3 web-based tools. All data was stored in a single postgresql database which itself was maintained as a separate app with migrations done using Rails.

There was nothing much of a project development process that the client enforced. He'll just mail or skype me the requirements/tasks, and I'd just ship them. But I did use Trello for capturing the new tasks and invited client to collaborate and check progress. It worked for us.


#### __How did your work impact the company, and was the client happy?__
The tasks I worked on for these tools directly affected the company's bottomline, as each one of these tasks increased the value proposition for the customer.

The CEO Mike appreciated the work and the timely delivery of the tasks done, regularly via skype.
