---
layout: post
read_time: true
show_date: true
title:  Allow almost non of the things
date:   2022-05-27 10:00:00 +1000
description: How do you get started with allow listing
img: posts/2022-05-27-app-allow-listing/hero.jpg
tags: [security,non-technical]
author: Peter Dodemont
---
Allow listing is probably the most effective way out there to stop malware in its tracks. With allow listing you configure which applications can be run, everything else will be blocked. This means that even if some malware found its way onto the device if it has not been previously allowed to run, it will not be able to run.\
But how do you even get started with it and how do you know what to block and what to allow. Those are some of the questions I hope to answer for you in this article.

## What is Allow Listing and How Does it Work
As mentioned in the intro allow listing works by defining what applications are allowed to run and prevent anything else from running. The way this is achieved is by way of an agent that runs on the device that will intercept any application launches. It will then validate that the application is authorized using the rules that are defined in its configuration. If a rule authorizing is found for the application it will be allowed to run, otherwise, it will be blocked.\
Defining what applications are allowed usually happens in 1 of 3 ways, using a file hash, using publisher information, or using file location. Each of these methods has pros and cons.
* File Hash

You can think of a file hash like a fingerprint for a file. A hashing algorithm is used to convert the data of the file into a series of letters and numbers. This series is always the same length no matter the size of the file and is (usually) significantly smaller than the original files. Crucially for a given algorithm a file with the same data will always produce the same hash and for all intents and purposes, files with different data will produce different hashes.\
File hashes are the most secure way of building an allow list. Since each file will have to match exactly with a previously hashed file. Unfortunately, unless you have a team dedicated to allow listing this will be nearly impossible to maintain as every file that gets changed (say after installing security updates) would need to be rehashed and added back to the list. This can amount to thousands of files depending on what you are updating.
* Publisher Information

Publisher information is a lot more useful in most scenarios as it allows you to use the details of the certificate that was used to sign the application for allowing an application. I won't go into details about how application signing works, all you need to know is that it requires a certificate that is normally purchased from a 3rd party certificate authority.\
Applications from the same companies are usually signed with the same certificates. This isn't always the case (especially for very large companies that develop lots of apps), but generally, even if it's not the same the number will be limited.\
When using publisher information, you are allowing all applications that have the same publisher information to be run. This is less secure than file hashes, but also comes with much less management overhead. Acquiring a certificate for publishing apps isn't overly difficult, but since each certificate has a thumbprint that uniquely identifies it, even a new certificate with identical information will not match the allowed publisher information as the thumbprints won't match. So the only real risk here is from certificates that have been stolen (which happens occasionally).
* Location

File location is the least secure way of doing allow listing. It is still better than not having it, but all that a malicious actor would need to do is move the malware to the allowed location and then the application can run. With it being so easily circumvented, you might wonder why it is an option at all. Sometimes applications are not signed and they might be updated frequently enough that using a hash isn't manageable. This is usually the case for internally developed apps, they are rarely signed and update frequency on them will be very inconsistent. Non-admin users can't copy files to every location (they can't copy to "Program Files" for example), so the risk when allowing those locations is less than one where everyone can write to.

That is, in a nutshell, how application allow listing works. Now that we know this let's move on to potential issues, my requirements, and my actual deployment process.

## Possible Pitfalls
Before I will delve into my process for getting allow listing deployed, I want to go through some of the possible issues that can arise if the deployment of allow listing isn't planned carefully. (This won't be an exhaustive list, just the ones I believe are the most important ones.)
* Excessive User Friction

This is probably the one that is the most obvious, but also the most important. As with any good security solution, user experience should be an important consideration. By its nature, allow listing will cause some user friction, as unsanctioned applications that users are running will get blocked. But the amount of friction can be reduced if the deployment is handled correctly and only the unsanctioned applications are blocked.
* Lost Productivity

This is another really important consideration. The majority of users should be able to be equally productive during and after the roll-out as they were before. Again planning is key to making sure this happens.
* Inadequate Communication

Users are going to have to be informed about what is going to happen as there most likely will be some sort of impact on them. The best communication is targeted at the right people and will also already answer some of the questions people will ask.
* Lack of Training and Support for the Support Staff

Inevitably users will run into problems and call the service desk for assistance, if the service desk agents are not properly trained, they could provide the wrong information to the user. By training them they will not only be able to provide users with the right information, but you might also be able to delegate some tasks to them meaning the users are assisted more quickly.

## My Requirements
Now that we know some of the potential issues, let's take a look at what requirements I put on allow listing software.
* Stacking Policies

What I mean by stacking policies is that you can have multiple policies apply to a device and it will combine all of the policies for you. This is super handy because it allows you to create a base policy for everyone, and then create smaller additional policies for certain groups of people that have special requirements (like the service desk or development team).
* Centralized Logs

The logs should all be collected and sent off to a central location from where you can look at them. Having to dig up the log from each individual machine will make it very difficult to get an accurate representation of all the applications being run efficiently.
* Easy to Configure Policies

It should only take a couple of minutes to be able to look at the log and add the item from the log to a policy. Ideally, you'll be able to add an application to a policy directly from the log.
* Cloud-Based

In the current environment, people work from anywhere and they might not always be connecting to a VPN, so having a management console in the cloud means that as long as there is internet connectivity policies can be updated on the clients. It doesn't have to be a SaaS solution, a self-hosted cloud-based system will work as well, but does usually require a lot more maintenance work, as you have to keep the underlying OS and the application itself up to date.
* Lightweight Agent

The agent will run on your clients whenever the client is on, so having a client that doesn't consume a lot of resources is critical to making sure the performance impact on the clients is minimal.
* Override Mode

Being able to temporarily override the enforcement is incredibly useful when doing application updates. It allows you to install the application on the device and then capture any executions that would be blocked. These can then immediately be added to the policies before the updates are rolled out to everyone else (this is especially useful for applications added via file hash). The override should only be temporary though and revert back to enforcement automatically after the set time expired. It should also be protected so it cannot just be enabled by anyone.

## My Deployment Process
If you have read some of my previous articles you'll know that I normally have 3 different groups of people, it is no different for this process.\
First, there is what I call the "IT Test Group". This is 1 or 2 people from the different teams in IT. It's important to not include an entire team in this group so as to not cause an entire team to be out of action in case of issues. I am usually not included as I will use myself for the initial configuration and testing before I start deploying anything.\
The second group I call the "Business Test Users". This is a group of people that represents a cross-section of the entire business. Generally, it is at least 1 user from each team in the business as well as a person from every single office type, if the business has multiple office locations (e.g. regional and metro).\
The final group of people is everybody else. This is not an actual group per se, but it is defined as all users in the business except those in the other 2 groups. It's important to note that it excludes the other 2 groups. Otherwise, you won't be able to follow the same process for any future changes if needed.\
Depending on the specific needs of the business, there will likely be some additional groups. Usually this will be specific groups of people within the business that have different application needs. For example IT support usually has tools related to account management the rest of the business won't need, or developers will need specialized tools for their development work.

Now that we have the groups, it's time to start the actual deployment process.\
After having done all the testing on myself or my test account, the first step is to deploy the agent to the IT test group and run it in audit mode (every allow listing solution I am aware of has the ability to run in audit mode). In audit mode, the agent will collect and log the executions without actually blocking them. With the agent deployed, it is time to check the logs every day for applications that have been run. They will all have a way to collect the logs, but how to check the logs will differ between solutions, so you will need to figure out the way to do this for your chosen solution.\
With the logs, I can start seeing applications that were run and can review each application and decide if it will be allowed. Some will be easy as they are part of your SOE or deployment platform. Some will not be, for those I will reach out to the service desk team. They will know what the application is used for in the business as they will have been fielding calls for it in the past. Once you have the information from the service desk you should be able to make a decision on most of the applications, but sometimes there are applications that they don't know about either. For those ones, it's best to reach out to the actual users themselves and get them to explain to you why they are using this application. Once it has been decided an application is allowed it needs to be added to the list using one of the methods mentioned before. Which method to choose for each file will depend on a number of factors. I will usually choose to use the publisher information method first as that offers a nice balance between security and manageability. But sometimes this is not possible (if for example, you want to allow Google Earth but not Google Chrome, since both have the same publisher information). In those cases, I will need to choose between file location or file hash. In general terms, I would only choose file location if the location is somewhere you can't just copy files to as a non-administrator (e.g. the "Program Files" folder), since that means an additional barrier that needs to be overcome by any malicious actor, and the application needs to that have frequent updates as well. If the application is updated infrequently or it is in a location where anyone can copy files to then a file hash is the better choice, as that means even if other files are copied in the same location they would not run.\
I will leave the audit mode enabled for the first group for several months (yes months), while monitoring the logs. The first few days will have lots to do, but that will go down quite quickly until hardly anything new pops up. The reason for leaving it for a few months is that there might be processes that only run once a month, once a quarter, or even once a year.\
Once there hasn't been any new application for a few weeks, I repeat the same process with the 2nd group of users, the "Business Test Group". The process is identical and also takes a few months. Usually, there will be a limited amount of new applications that pop up, but it's better to be safe.\
Once the business test users have had it for a few months, I will move the IT team to enforced mode while also putting the final group (being everyone else) into audit mode.\
Normally there will be hardly any new items that pop up either from the enforced IT team or the rest of the business, so I usually only leave it a couple of weeks before enforcing allow listing on the business test users. Once again there will be very few applications that pop up, and if they do they are usually very minor. Then a few weeks later it is time to enable enforcement for the rest of the users. By this stage, there should be no new apps popping up anymore, unless a new app is being rolled out.

During the deployment process communication is very important. Since applications will stop working once you go to enforce mode it is best to let each group of users know what to expect a few days before enabling the enforcement. There is no need for communication during the audit mode as that should be totally transparent to the users (and sending too much communication is just as bad as not enough).\
Another important part is to learn from the process and adjust the communication accordingly. For example, you might notice there are a lot of people running Google Chrome even though the sanctioned browser is Microsoft Edge. In that case, it will be useful to mention in the communication that Chrome will stop working when allow listing comes in and that they should switch to Edge now and report any issues they have with it. This will reduce the number of calls coming into the service desk at enforcement time, but it will also give the service desk something to refer to when questioned about it.\
The communication should also include a way for people to request applications be added to the allow list if they can provide a valid justification. And the first few weeks, responses to those queries should be handled as a priority.

That is how I go about deploying application allow listing. This has proven to be a mostly smooth process with minimal user friction for users using allowed apps. I didn't mention any specific products in the article, but if you are interested in knowing what product I recommend, reach out to me and I can give you a breakdown of the products I have used and why I like some more than others.\
And as always if you have any questions or comments, please reach out.