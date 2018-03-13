---
layout: page
---

([**portfolio**](/portfolio/) > **ubertor**)


## Ubertor RealEstate Website Development


#### __When did you work on this project?__
4 months in 2014 from April.


#### __What is this project about?__
The client's main product is an online website builder for real-estate companies. They built it with PHP and wanted the whole project to be transitioned to Rails 3 (at the time). The app also had many pages that were slow to load which need to be optimised.


#### __Who was the client?__
[__Ubertor__](http://ubertor.com/)


#### __What did _you_ do exactly?__
I was the sole developer in this project. I spent a few initial days to get acquainted with PHP and the database structure. Then I used the [rails best practices](https://github.com/railsbp/rails_best_practices) gem to refactor the changes made by the previous developer.

Then it was time to focus on the slow queries. I used mysql's EXPLAIN PLAN to see if the queries were using the appropriate indices and added/removed proper indices based on the query results. I was able to bring down the running time of many queries from many minutes to a few seconds. This improved the UX of many of the pages.


#### __Process/Technologies Used__
* Rails 3, Ruby 1.9.3, MySql
* Capistrano for deployment


#### __How did your work impact the company, and was the client happy?__
The PHP version of the app was bringing in business for the client for more than 5 years before it came to us.

The necessary cleanup that I did - Rails + mysql optimisations - was much appreciated by the client as it brought a new and better experience to their users, which directly impacted positively their business.
