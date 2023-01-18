---
title: 'An Empirical Study of Performance in Laravel Applications'
date: '2019-01-11T00:02:10-05:00'
author: Adrien Poupa
url: /empirical-study-of-performance-in-laravel-applications/
categories:
    - PHP
tags:
    - eloquent
    - Laravel
    - orm
    - performance
---

As part of my [Software Verification and Testing](https://petertsehsun.github.io/soen7481/) class at Concordia University, my team and I chose to study empirically the performance of some popular Laravel applications. The goal was to replicate the paper [“How not to structure your database-backed web applications: a study of performance bugs in the wild”](http://hyperloop.cs.uchicago.edu/220-HowNotStructure.pdf) by Junwen Yang et al.

This paper, focused on the Ruby ecosystem, found that ORM API misuses such as inefficient queries, lack of pagination, inefficient eager loading or lazy loading were common causes for performance degradation. They also found that databases exhibited some design problems such as missing fields or indexes. To find such defects, they identified 12 popular Ruby applications and filled them with dummy data – 200 records, 2,000 and 20,000 records. They recorded performance issues, applied fixes and compared the results before and after applying them.

Thus, to apply a similar approach to the PHP ecosystem, we chose [Attendize](https://github.com/Attendize/Attendize), [Cachet](https://github.com/CachetHQ/Cachet) and [Monica](https://github.com/monicahq/monica) as they are popular open-source Laravel applications. Attendize is a ticket selling and event management platform, Cachet is a status page system and Monica is a personal CRM.

First, let’s seed the databases: insert 200 records, 2,000 and 20,000 records for each application. We insert such records in the most significant tables (contacts for Monica for example). To do so, we used the seeders bundled with the applications when they were available, we [fixed them](https://github.com/monicahq/monica/pull/2099) to insert more data if need be, or we created them if they did not exist. As expected, the size of the databases grows linearly.

![](https://cdn.poupa.net/uploads/2019/01/dbsize.png)<figcaption>Size of the database for each project</figcaption>Then, to evaluate the performance of the applications, we used the [Laravel Debugbar](https://github.com/barryvdh/laravel-debugbar) and [Laravel Telescope](https://github.com/laravel/telescope). They allow seeing the database queries, the time they took and so on. More interestingly, the DebugBar counts the number of unique queries and duplicated queries. We will focus on reducing the latter.

![](https://cdn.poupa.net/uploads/2019/01/debugbar-1024x220.png)<figcaption>The Laravel DebugBar in action</figcaption>For the most important pages, we counted the number of unique queries and compared it against the number of duplicated queries for the three applications, before and after we applied our fixes for the three databases.

![](https://cdn.poupa.net/uploads/2019/01/attendizequeries-1024x502.png)<figcaption>Number of queries for Attendize</figcaption>![](https://cdn.poupa.net/uploads/2019/01/cachetqueries-1024x507.png)<figcaption>Number of queries for Cachet</figcaption>![](https://cdn.poupa.net/uploads/2019/01/monicaqueries.png)<figcaption>Number of queries for Monica</figcaption>We were surprised to see such large numbers of duplicated queries in popular applications. This can be explained by the easiness to forget that every call to Laravel’s ORM, Eloquent, actually translates to a database query.

To reduce the number of duplicated queries, we applied several different strategies. First, we used the DebugBar to identify which queries were duplicated, then, either we assigned the result of the query to a local variable or an attribute, or we used eager loading to solve the [N+1 problem](https://laravel.com/docs/5.7/eloquent-relationships#eager-loading).

![](https://cdn.poupa.net/uploads/2019/01/variablefix.png)<figcaption>Assigning the result of a query to a local variable in Attendize</figcaption><figure class="wp-block-image">![](https://cdn.poupa.net/uploads/2019/01/eagerload.png)<figcaption>Eager loading the User model in Cachet</figcaption>Another common issue we found was the lack of pagination. In Monica, the contact page would display all the contacts regardless of the database size. This is becoming a performance hog as the database grows as shown in the following figure. We measured the loading time with [jMeter](https://jmeter.apache.org/) to avoid the overhead added by the DebugBar.

![](https://cdn.poupa.net/uploads/2019/01/loadingtimemonica.png)<figcaption>Average loading time in seconds for the two pages  
 as measured by jMeter for Monica</figcaption>The difference in terms of loading time [once pagination was added](https://github.com/monicahq/monica/pull/2135) with vuejs is impressive. The loading time remains constant regardless of the database size.

To conclude, the applications that we studied showed similar results to those studied in the Ruby ecosystem; it is usually possible to greatly improve the performance of a Laravel application by paginating the results returned by the database, not calling the same Eloquent model a few lines apart and being aware of the N+1 problem.

The full report [can be found here](https://cdn.poupa.net/uploads/2019/01/LaravelPerformanceStudyReport.pdf). This [GitHub repository](https://github.com/AdrienPoupa/soen7481-project) contains the results of our experiments, including the databases should one want to replicate our study. Feel free to contact me if you have any questions.