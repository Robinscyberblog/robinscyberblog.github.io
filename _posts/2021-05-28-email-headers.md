---
layout: post
title:  "Email Header Analysis"
date:   2021-05-16 14:30:06 -0500
categories: jekyll update
---
In this post I want to go over simple email header analysis using the Received: lines. While not very glamorous, email header analysis is a very practical skill to hone. It can even be kind of fun to investigate an email to try to determine whether or not it is legitimate. So, what is an email header? An email header is a section of code that identifies an email’s details and travel history. To begin your investigation, you’ll need to know what you are looking for when you pull the email header. 

![Full View Header](https://robinscyberblog.github.com/images/Screenshot_full.png) "Full View Header")

First, you’ll want to focus your attention on the Received: headers. These lines are important, because they show the path a message takes from the initial sender to the receiver. If there are several **Received:** headers, the message was handled by multiple servers. It is also important to note that these headers are read in reverse order; from the bottom (where the message originated) to the top (where the message was finally delivered). 
![Full View Header](Screenshot_full.PNG) "Full View Header")
Let’s take a look at an email I received that is pretending to be from eHarmony.com.

You can see in the first **Received:** header, it shows the initial message came from 2602fed2730003481616b16f00000001.kfo9rcm.ga and was received by mx.google.com. You will also see the IP address, in this case an IPv6 address, in brackets. 

With this information, I can make some inferences about whether or not this email came from eHarmony. First and foremost, it seems highly unlikely that eHarmony would use an address like 2602fed2730003481616b16f00000001.kfo9rcm.ga in their email infrastructure. Furthermore, if I conduct a DNS lookup of eHarmony.com, the IP address associated with the email I received is not included in the IP addresses resolved for eHarmony. In fact, the IP address is associated with a host out of Armenia (also a red flag).


These are some good indicators that the email is not legitimate. There are other mechanisms you can check for validity such as SPF and DKIM signatures, but for this email they were not useful because the email didn’t even attempt to spoof the eHarmony.com domain. Rather, they simply included "eHarmony Offer" in the subject line. You can see in this screenshot, the email already seems suspicious just by looking at it in Gmail. 
