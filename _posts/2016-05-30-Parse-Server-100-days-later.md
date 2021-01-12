---
layout: post
author: michal
title: \#16 Parse Server – 100 days later
excerpt: 
---
Around 100 days ago, Parse.com announced its [shut down](http://blog.parse.com/announcements/moving-on/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) on 28th of January 2017. At the same time an open sourced [Parse Server](http://blog.parse.com/announcements/introducing-parse-server-and-the-database-migration-tool/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) was born. Team advised to migrate databases until the end of April 2016 and to move services to self hosted instances by the end of June 2016. By following this guidance developers are to avoid procrastination and have plenty of time for transition and pointing their apps to new endpoints. 

#### Community

Facing additional work, limited features and additional costs, some may have left Parse.com platform. However, many saw an opportunity in a newborn Parse Server. Blogs started to write extensive tutorials about migration, missing features and ideas for the future of the platform. 
We did not give up on Parse Server either. In [issue \#4](https://swifting.io/blog/2016/02/08/4-migrartion-from-parse-to-heroku/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post), we have covered a migration from Parse.com to Heroku and mLab Mongo as a service provider. 

![Parse Server github](https://raw.githubusercontent.com/swiftingio/blog/%2316-Parse-100-days-later/%2316%20Parse%20100%20days%20after/parse-server-github.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

[Github repository](https://github.com/ParsePlatform/parse-server?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) is the strongest evidence of how big and active community of Parse Server have become. At the time of writing it has more than 9k of stars, more than 2k forks and more than 1k of commits from 70+ contributors. Not only missing features are being added but also features not existing in Parse.com like [Live Queries](http://blog.parse.com/announcements/parse-server-goes-realtime-with-live-queries/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

![Parse.com and Parse-Server comparison](https://raw.githubusercontent.com/swiftingio/blog/%2316-Parse-100-days-later/%2316%20Parse%20100%20days%20after/blog-table-comparison.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

I really like a little table Fosco Marotto placed in his [post](http://blog.parse.com/announcements/what-is-parse-server/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) that introduced Parse Server. Yes we have to sacrifice convenience, but there is much more to gain.

All this makes us believe that Parse Server is here to stay and prosper. In this post we will write about current status of Parse Server, choices you have while migrating your app or setting up a clean project and what to be careful of.

#### It's all about that... base
Whether migrating from an existing Parse.com deployment or setting up a clean project on Parse Server you need to take care about two major components: a solid MongoDB database and a node.js hosting. Below are results of our research and experiences. 

Let us know in comments what you have been looking at and what powers your Parse Server setup.

##### mLab.com
![mLab.com](https://raw.githubusercontent.com/swiftingio/blog/%2316-Parse-100-days-later/%2316%20Parse%20100%20days%20after/mlab.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

[mLab](https://mlab.com/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post), formerly MongoLab, is the first you have probably heard of. It has a nice free tier of 500MB - can have many of these. They are great for development and testing. On production you would rather want to enable SSL, and have a Replica Set Cluster. 
SSL costs extra $80, but can be set account wise. Cluster can easily host multiple databases depending on your needs. You have choices of AWS, Amazon and Google platform in multiple regions. Nice interface can setup everything just by clicking. Good and responsive support team too. This is actually our choice.

##### Scalegrid.io
![Scalegrid.io](https://raw.githubusercontent.com/swiftingio/blog/%2316-Parse-100-days-later/%2316%20Parse%20100%20days%20after/scalegrid.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

We have considered [Scalegrid.io](https://scalegrid.io/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) for quite some time and actually went for a trial. They allow you to host on AWS, Azure, Digital Ocean, Bare metal, Joyent and can setup on your AWS machines too. There is more, like private clouds and Amazon VPC. Good and responsive support, really open to customizations, when asked. Competitive pricing and SSL comes at no extra charge. There is a 20% discount offer for a long term, reserved instances.


##### Compose.io
![Compose.io](https://raw.githubusercontent.com/swiftingio/blog/%2316-Parse-100-days-later/%2316%20Parse%20100%20days%20after/compose.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

[Compose.io](https://compose.io/mongodb?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post), formerly MongoHQ which is an IBM company. They have more broad spectrum of services and hosted MongoDB is just one of them. We saw there WiredTiger technology as an option first. WiredTiger is a technology that decreases usage of storage even 10x and boosts performance even 50-fold. It was actually used by Parse.com since some time. That is why they asked you to reserve 10x more space when migrating your DB expecting you to pick standard MMAP technology. Compose uses replica set by default and their pricing model is per GB. RAM memory and disk IOPs scale proportionally as you add more GBs. Looks really solid, however their pricing model and a lack of dedicated machines caused that we did not went for it.

##### ObjectRocket.com
![ObjectRocket.com](https://raw.githubusercontent.com/swiftingio/blog/%2316-Parse-100-days-later/%2316%20Parse%20100%20days%20after/objectrocket.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

[ObjectRocket](http://objectrocket.com?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) is the last of hosted MongoDB as a service providers that we have looked at. ObjectRocket is a Rackspace company. As it is at Compose.io, MongoDB is not their only offering. They offer Sharded Clusters and WiredTiger engine. In comparison to other vendors they were more expensive without showing clear advantages. 


##### MongoDB Cloud Manager
![MongoDB Cloud Manager](https://raw.githubusercontent.com/swiftingio/blog/%2316-Parse-100-days-later/%2316%20Parse%20100%20days%20after/mongocloud.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

[MongoDB Cloud Manager](https://www.mongodb.com/cloud?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) is a different kind of a beast. First, it is provided by MongoDB itself. Second, it is used to deploy and manage MongoDB cluster on your own AWS machines or servers. You setup agents or give Cloud Manager access (provisioning) to your AWS account. It seems a really great way to have more control over your cluster and is a very cost effective path to follow if you have multiple deployments. As it usually is, these benefits come at cost. You will spend more time setting up and maintaining your deployment. Still, we consider it as a future option for our growing number of Parse Server backed apps.

##### Your own deployment

It is possible that you are MongoDB Jedi, or you can use a help of one, then you may decide to setup your cluster all by yourself. This will require some time for setup and maintenance plus knowledge. Two major benefits are cost effectiveness and full control. You can find some starting point links in references section. 

For development and playing around you may set a single Mongo process where your Parse Server is hosted. However, this is a bad decision - your server and database will share resources, which is not scalable. If your hosting goes down, then your whole setup goes down. Thus, if your data can be hosted in cloud and you are not limited in other ways, reconsider one of nice trials and free tiers (ex. mLab's 500MB). 

To sum up this section, I was really tempted to create a table with a full-blown comparison, but that would just quickly outdate 😉.

#### A new home
Finding a good hosting for node.js deployment of Parse Server is not a big deal. There are plenty of platforms that support it. Topic is also less frightening. Most of developers already have experiences with at least one of PaaS providers. You can setup Parse Server on bare machines like Amazon EC2 or use PaaS like Heroku or AWS Elastic Beanstalk where node.js environment is just ready for your package. 

***NOTE:*** One thing to check for when choosing a smaller or a less known provider - make sure that all [Parse Server dependencies](https://github.com/ParsePlatform/parse-server/wiki/Parse-Server-Guide?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#prerequisites) are in required versions. Keep an eye on node.js, python and MongoDB versions support.
Back then, shortly after Parse Server was born, I was even preparing a tutorial about setup on one of smaller PaaS providers. After couple of hours of work I got stuck because of lack of support for a required MongoDB version 😓. Hopefully, those days are far behind us.

##### Where can I deploy?
[Parse Server github page](https://github.com/ParsePlatform/parse-server?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#parse-server-sample-application) contains a list of providers that we can assume are recommended and easiest to deploy. Heroku and AWS even have these cute "Deploy to Heroku" and "Deploy to AWS" buttons that get you half way through with one click.

![Deploy to Heroku](https://www.herokucdn.com/deploy/button.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) ![Deploy to AWS](http://d0.awsstatic.com/product-marketing/Elastic%20Beanstalk/deploy-to-aws.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) 

Clearly you are not limited to choices above. It should run well on bare machines (ex. Linux), virtual machines or whatever you have available for your personal projects or at your company.

##### How to pick a hosting for myself?
Going through comparison of even only major platforms would be duplicating of work. Make sure to see the list on Parse Server Github link above. If you have some prior experiences with one of platforms, go for it. If you are fresh go for Heroku or AWS - they have a free tier and are easy to setup.

We have had some prior experiences with Heroku and AWS. Our choice was AWS Elastic Beanstalk. So far we enjoy it. 

#### Security of my deployment

That is quite a topic, probably for another post. Now, as you are in charge of your mBaaS, it is worth at least a short note. Here are a couple of things to look at:

- Make sure that all your connections are encrypted. With free certificates from [Let's Encrypt](https://letsencrypt.org/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) there is no complaining that it costs extra.
- All connections are all. Connection to your MongoDB should also be encrypted. Look for 'ssl=true' param in connection string.
- Use Replica Sets/Sharding on your MongoDB - to duplicate and/or split data across machines.
- Tighten access to your MongoDB deployment - set IP rules or Security Groups, so that only certain machines can access it.
- Take a look at [Advanced Options](https://github.com/ParsePlatform/parse-server?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#advanced-options) of Parse Server - disable anonymous users and client class creations.
- Make yourself familiar with ACL.
- In your cloud code you can check for session, send 'X-Parse-Session-Token' from your client.
- You are a master of your MasterKey. Protect it.
- Avoid hardcoding keys and IDs in server code, use [environment variables](https://github.com/ParsePlatform/parse-server?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#using-environment-variables-to-configure-parse-server).
- Keep your Parse Server version up-to-date. In this way you get not only new and cool features but also security patches.

Would you like to share some security tips? Let us know in comments or on [Twitter](https://twitter.com/swiftingio?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

#### Features

When Parse Server was announced, we have all seen that it lacked many features that we have been used to in Parse.com. Not all of them are present at the moment, but major ones have been added really fast. With frequent, approximately bi-weekly releases, we can expect more to come. Actually, there are features like LiveQurery, numerous oAuth providers or File Adapters not present in commercial Parse.com.

What is included in Parse Server (at the time of writing):

- REST API,
- iOS, Android, JS, .NET and PHP SDKs,
- Cloud Code (though you may need some modifications while migrating),
- Basic iOS and Android [Push](https://github.com/ParsePlatform/parse-server/wiki/Push?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) Notifications,
- [oAuth authentication](https://github.com/ParsePlatform/parse-server/wiki/OAuth?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post): Twitter, Meetup, LinkedIn, Google, Instagram, Facebook and custom,
- Parse [LiveQuery](https://github.com/ParsePlatform/parse-server/wiki/Parse-LiveQuery?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) based on web sockets,
- File Adapters: GridStore(default), S3, Google Cloud Storage,
- Emails: via [Mailgun](https://github.com/ParsePlatform/parse-server?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#email-verification-and-password-reset), [Sendgrid](https://www.npmjs.com/package/parse-server-sendgrid-adapter?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) or [Mandrill](https://github.com/back4app/parse-server-mandrill-adapter?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- Global Config,
- Class Level Permissions,
- Parse-Dashboard (look in section below)

What is still missing:

- Analytics,
- In-App Purchases,
- [Background Jobs](https://github.com/ParsePlatform/parse-server/wiki/Compatibility-with-Hosted-Parse?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#jobs) (although you can run some on your node.js),
- [Webhooks](https://github.com/ParsePlatform/parse-server/wiki/Compatibility-with-Hosted-Parse?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#webhooks),
- Dashboard with full capabilities,
- [Multiple application](https://github.com/ParsePlatform/parse-server/wiki/Compatibility-with-Hosted-Parse?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#single-app-aware) support

![Parse Server changelog](https://raw.githubusercontent.com/swiftingio/blog/%2316-Parse-100-days-later/%2316%20Parse%20100%20days%20after/changelog.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

One more thing. It is always worth digging in [changelog](https://github.com/ParsePlatform/parse-server/blob/master/CHANGELOG.md?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) on Parse Server's git repository 😉. [Wiki](https://github.com/ParsePlatform/parse-server/wiki/Compatibility-with-Hosted-Parse?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) on compatibility article may also help.

#### Tutorials
Parse.com team really cares about your migration to Parse Server going as smooth as possible. You can see official blog posts presenting major features being added. On April 18th, a set of [video tutorials](http://blog.parse.com/learn/parse-server-video-series-april-2016/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) covering topics like environment setup, database migration, launching Parse Server app or things to look at when migrating your Cloud Code. This is on top of contents of Github repository and community blog posts.

![Parse Server Video Series](https://raw.githubusercontent.com/swiftingio/blog/%2316-Parse-100-days-later/%2316%20Parse%20100%20days%20after/videos-blog.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)



#### Cloud code migration

What I really liked about video series was the [Cloud Code Tips](https://www.youtube.com/watch?v=wAQsY0XdatQ&utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) clip. It starts with 2 tips about using modern JS for a cleaner code:

- Using [Promises](http://blog.parse.com/learn/engineering/whats-so-great-about-javascript-promises/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) instead of callbacks:

![Promises Parse blog](https://raw.githubusercontent.com/swiftingio/blog/%2316-Parse-100-days-later/%2316%20Parse%20100%20days%20after/promise.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

- Using [Arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post):

![Arrow Functions Mozilla](https://raw.githubusercontent.com/swiftingio/blog/%2316-Parse-100-days-later/%2316%20Parse%20100%20days%20after/arrow.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)


Third tip, probably the most important for people migrating, is about current user scope. This scope may be used for operations related to particular user or assuring security. In Parse.com you would use:

```
Parse.User.current()
```
That is no longer valid in Parse Server. Now you have to take care about passing user session yourself. You can do it either by using ```sessionToken``` parameter in operation or setting ```'X-Parse-Session-Token'```. Once you have taken care of passing session, then it is available in your Cloud Code as:

```
request.user
```
Specifically, session token is available in:

```
request.user.getSessionToken()
```

You can see more about handling session on [wiki](https://github.com/ParsePlatform/parse-server/wiki/Compatibility-with-Hosted-Parse?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#no-current-user) page.

#### GUI
Parse Server did not have an easy start. Developers immediately noticed lack of GUI for managing data, logs and other mBaaS components. Suddenly we have all appreciated the convenience of Parse.com 😁.

##### Adminca.com
This could not last long however. Brave and young team from [Adminca.com](http://adminca.com/blog?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) created a first GUI and made it available to community on March 3rd.

![Adminca](https://raw.githubusercontent.com/swiftingio/blog/%2316-Parse-100-days-later/%2316%20Parse%20100%20days%20after/adminca.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

With Adminca.com you can:

- Quickly attach your Parse Server deployment,
- Add and remove fields and classes,
- Add, remove and edit rows,
- Add collaborators,
- Export data to CSV,
- Attach multiple Parse Server deployments

Great Adminca's Team, facing the better-supported and incoming Parse Dashboard, froze further enhancements to their GUI. It is still a decent option for a free and hosted simple Parse Server data management.

##### Parse Dashboard

Parse Dashboard is a separate [project](https://github.com/ParsePlatform/parse-dashboard?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) on Github. You can run it on your localhost, on the same server as your Parse Server or on an entirely different machine.  It was announced on March 4th and contained very basic set of features. Soon, major features were added and it currently covers majority of our needs.

With Parse Dashboard you can:

- Add and remove fields and classes,
- Add, remove and edit rows,
- Setup Class Level Permissions (CLPs),
- Display Logs,
- Create Config,
- Configure API,
- Send push notifications,
- Attach multiple Parse Servers deployments

![Parse Dashboard](https://raw.githubusercontent.com/swiftingio/blog/%2316-Parse-100-days-later/%2316%20Parse%20100%20days%20after/parse-dashboard.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Parse Dashboard is doing well. With 1.5k stars and almost 40 contributors we can expect it to be well maintained and growing. 

Everything is well documented on Github page, so I would just want to attract your attention to security of your dashboard. You can setup Basic Auth credentials for access. For this to be safe is has to be accompanied by https protocol. Luckily, if you try to access it without https while not running on localhost you will not be let in. Good job team 👏 for protecting us from ourselves! 

Usually your load balancer on PaaS will provide SSL and that is enough to ensure secure connection. However, if ensuring secure connection is up to you, here is a useful snippet (Parse Dashboard documentation includes only http Express setup):

```
var express = require('express');
var ParseDashboard = require('parse-dashboard');
var fs = require('fs');
var https = require('https');
var key = fs.readFileSync('key.pem','utf8');
var cert = fs.readFileSync('cert.pem','utf8');
var credentials = {key:key, cert:cert};

var dashboard = new ParseDashboard(
{
"apps":[
{
"serverURL":"YOUR_SERVER_URL",
"appId":"YOUR_APP_ID",
"masterKey":"YOUR_MASTER_KEY",
"appName":"YOUR_APP_NAME"
}
],
"users":[
{
"user":"YOUR_DASHBOARD_USER_NAME",
"pass":"YOUR_DASHBOARD_PASSWORD",
"apps":[{"appId":"YOUR_APP_ID"}]
}
]
});

var app = express();

// make the Parse Dashboard available at /dashboard
app.use('/dashboard', dashboard);

var httpServer = require('https').createServer(credentials, app);
httpServer.listen(443);
```

**UPDATE:**

Parse Dashboard since version 1.0.11 allows you to specify ```--sslCert``` and ```--sslKey``` params to supply paths to certificate and key. They can also be set as node.js environment variables ```PARSE_DASHBOARD_SSL_CERT``` and ```PARSE_DASHBOARD_SSL_KEY``` respectively. You can lookup changes in [this](https://github.com/ParsePlatform/parse-dashboard/commit/561196f84eb27c778b63b0f3f6b03297d6eca4c3?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) commit. It has a similar logic to our snippet above.

#### Adapters

Last thing to cover in this post is an extensibility of Parse Server. There is a less known space for community projects like modules and adapters that are not part of core Parse Server. It is called [parse-server-modules](https://github.com/parse-server-modules?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) and already contains some cool stuff:

![Parse Server modules](https://raw.githubusercontent.com/swiftingio/blog/%2316-Parse-100-days-later/%2316%20Parse%20100%20days%20after/modules.png?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

#### Summary and references
To sum up, Parse Server is here to stay. For our team it already contains most important features. Community support is vast and strong. Yes, we have had to sacrifice quite a bit of convenience but a little of JS has never killed anybody. Actually, we have got more awareness of what is powering our mobile apps. Initially, as probably many of you, we were afraid where to go with our Parse.com deployments. Right now we just go migrating to self hosted Parse Server with our existing projects one by one. At least until something better pops up on horizon 😆.

Hopefully, you have enjoyed our summary. We would love to see your thoughts, experiences and tips! 
Let us know in comments or on [Twitter](https://twitter.com/swiftingio?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).


Finally, it would not be us if we did not provide a proper pack of references. Here they come:

###### Parse Server
- [Parse Server Github repository](https://github.com/ParsePlatform/parse-server?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) 
- [Parse.com blog post introducing Parse Server
](http://blog.parse.com/announcements/what-is-parse-server/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) 
- [Great Parse Server Video Series](http://blog.parse.com/learn/parse-server-video-series-april-2016/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Swifting.io issue \#4](https://swifting.io/blog/2016/02/08/4-migrartion-from-parse-to-heroku/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)


###### Databases
- [mLab.com (MongoLab)](https://mlab.com/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Scalegrid.io](https://scalegrid.io/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Compose.io (MongoHQ)](https://compose.io/mongodb?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [ObjectRocket](http://objectrocket.com?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [MongoDB Cloud Manager](https://www.mongodb.com/cloud?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Custom MongoDB Sharded cluster #1](https://docs.mongodb.com/v3.0/core/sharded-cluster-architectures-production/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) and [\#2](https://docs.mongodb.com/v3.0/tutorial/deploy-shard-cluster/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Custom MongoDB Replica Set](https://docs.mongodb.com/v3.0/tutorial/deploy-replica-set/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) and [Replica Set Tutorials](https://docs.mongodb.com/manual/administration/replica-sets/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Mongo RocksDB - for performance and capacity](https://github.com/ParsePlatform/parse-server/wiki/MongoRocks?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

###### Parse Server hosting and security
- [Deploying on Heroku](https://devcenter.heroku.com/articles/deploying-a-parse-server-to-heroku?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Deploying on AWS Elastic Beanstalk](http://mobile.awsblog.com/post/TxCD57GZLM2JR/How-to-set-up-Parse-Server-on-AWS-using-AWS-Elastic-Beanstalk?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Deploying on Microsoft Azure](https://azure.microsoft.com/en-us/blog/azure-welcomes-parse-developers/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Deploying on Google Cloud](https://medium.com/google-cloud/deploying-parse-server-to-google-app-engine-6bc0b7451d50?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#.ut4u12zih)
- [Other providers](https://github.com/ParsePlatform/parse-server?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#parse-server-sample-application)
- [Parse Server user handling](https://github.com/ParsePlatform/parse-server/wiki/Compatibility-with-Hosted-Parse?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#no-current-user)
- [Parse Server and environment variables](https://github.com/ParsePlatform/parse-server?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#using-environment-variables-to-configure-parse-server)
- [Let's Encrypt - free SSL certificates](https://letsencrypt.org/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Stanislav Sidelnikov post on Security Considerations](https://www.stansidel.com/2016/03/parse-server-security-considerations-and-server-updates/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

###### Fully hosted Parse Server (UPDATE: 10.10.2016)
- [NodeChef](https://nodechef.com/parse-server?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Back4App](https://www.back4app.com?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Oursky](http://parse-hosting.oursky.com?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Sashido](https://www.sashido.io?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

###### Parse Server features
- [Parse Server changelog](https://github.com/ParsePlatform/parse-server/blob/master/CHANGELOG.md?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Live Queries](http://blog.parse.com/announcements/parse-server-goes-realtime-with-live-queries/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) and [wiki](https://github.com/ParsePlatform/parse-server/wiki/Parse-LiveQuery?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [File adapters](http://blog.parse.com/announcements/hosting-files-on-parse-server/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) and [wiki](https://github.com/ParsePlatform/parse-server?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#configuring-file-adapters)
- [Push notifications](http://blog.parse.com/announcements/parse-server-push-notifications/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) and [wiki](https://github.com/ParsePlatform/parse-server/wiki/Push?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [3rd party oAuth](https://github.com/ParsePlatform/parse-server/wiki/OAuth?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Wiki article on compatibility](https://github.com/ParsePlatform/parse-server/wiki/Compatibility-with-Hosted-Parse?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

###### GUI
- [Adminca.com](http://adminca.com?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) 
- [Parse Dashboard](https://github.com/ParsePlatform/parse-dashboard?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

###### Other
- [Early day Parse.com and Parse Server comparison](http://savvyapps.com/blog/parse-server-missing-features-solutions-migration?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [RayWenderlich.com Parse Server Tutorial](https://www.raywenderlich.com/128313/parse-server-tutorial?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Parse Drops the BaaS – Under The Bridge](http://www.alexcurylo.com/2016/01/31/parse-drops-the-baas/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
