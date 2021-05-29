---
layout: post
title:  "Email Header Analysis"
date:   2021-05-16 14:30:06 -0500
categories: jekyll update
---
In this post I want to go over simple email header analysis using the **Received**: lines. While not very glamorous, email header analysis is a very practical skill to hone. It can even be kind of fun to investigate an email and try to determine whether or not it is legitimate. So, what is an email header? An email header is a section of code that identifies an email’s details and travel history. To begin your investigation, you’ll need to know what you are looking for when you pull the email header. 

First, you’ll want to focus your attention on the **Received:** headers. These lines are important, because they show the path a message takes from the initial sender to the receiver. If there are several **Received:** headers, the message was handled by multiple servers. It is also important to note that these headers are read in reverse order; from the bottom (where the message originated) to the top (where the message was finally delivered). 

Let’s take a look at an email I received that is pretending to be from eHarmony.com.

![image](https://user-images.githubusercontent.com/84248865/120082323-ed57bc00-c087-11eb-8143-5bb11ac92e60.png)

You can see in the first **Received:** header, it shows the initial message came from 2602fed2730003481616b16f00000001.kfo9rcm.ga and was received by mx.google.com. You will also see the IP address, in this case an IPv6 address, in brackets. 

![image](https://user-images.githubusercontent.com/84248865/120082464-e54c4c00-c088-11eb-94ae-7a2d3b049fe7.png)

With this information, I can make some inferences about whether or not this email came from eHarmony. First and foremost, it seems highly unlikely that eHarmony would use an address like 2602fed2730003481616b16f00000001.kfo9rcm.ga in their email infrastructure. Furthermore, if I conduct a [DNS lookup](https://kb.intermedia.net/Article/819) of eHarmony.com, the IP address associated with the email I received is not included in the IP addresses resolved for eHarmony. In fact, the IP address is associated with a host out of Armenia (also a red flag).

![image](https://user-images.githubusercontent.com/84248865/120082536-378d6d00-c089-11eb-8ce9-45ace4c0d92a.png)

These are some good indicators that the email is not legitimate. There are other mechanisms you can check for validity such as [SPF and DKIM](https://woodpecker.co/blog/spf-dkim/), but for this email they were not useful because the sender didn’t actually attempt to spoof the eHarmony.com domain. Rather, they simply named the account "eHarmony Offer" to try to trick the recipient. As you can see in this screenshot, the email already seems suspicious just by looking at it in Gmail. 

![image](https://user-images.githubusercontent.com/84248865/120082735-34df4780-c08a-11eb-9cfc-9b57a7bf3748.png)

Now, give it a try on your own! See what you can learn by looking at the **Received:** headers of a suspicious email. Just be sure to do it carefully and do not analyze the email directly through the [email client](https://help.returnpath.com/hc/en-us/articles/220569547-What-is-a-Mail-User-Agent-MUA-), in case of malicious content. For a more in-depth analysis tutorial, check out this [video](https://www.youtube.com/watch?v=nK5QpGSBR8c) by 13Cubed. It's very informative and really helped me understand the process.

Thank you for reading!