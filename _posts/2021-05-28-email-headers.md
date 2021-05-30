---
layout: post
title:  "Email Header Analysis"
date:   2021-05-16 14:30:06 -0500
categories: jekyll update
---
In this post I want to go over email header analysis using the **Received**: lines. While not very glamorous, email header analysis is a very practical skill to hone. It can even be kind of fun to investigate an email and try to determine whether or not it is legitimate. So, what is an email header? An email header is a section of code that identifies an email’s details and travel history. To begin your investigation, you’ll need to know what you are looking for when you pull the email header. 

First, you’ll want to focus your attention on the **Received:** headers. These lines are important, because they show the path a message takes from the initial sender to the receiver. If there are several **Received:** headers, the message was handled by multiple servers. It is also important to note that these headers are read in reverse order; from the bottom (where the message originated) to the top (where the message was finally delivered). 

Let’s take a look at an email I received that is pretending to be from eHarmony.com. You can view the full header as a .txt document [here](https://raw.githubusercontent.com/Robinscyberblog/robinscyberblog.github.io/master/Untitled%20document.txt). 

![image](https://user-images.githubusercontent.com/84248865/120083986-37459f80-c092-11eb-8505-f3219c69ee77.png)

You can see in the first **Received:** header, it shows the initial message came from 2602fed2730003481616b16f00000001.kfo9rcm.ga and was received by mx.google.com. You will also see the IP address, in this case an IPv6 address, in brackets. 

![image](https://user-images.githubusercontent.com/84248865/120119554-f3b86780-c15d-11eb-8252-6ff6452526c8.png)

With this information, I can try to determine whether or not this email came from eHarmony. First and foremost, it seems highly unlikely that eHarmony would use an address like 2602fed2730003481616b16f00000001.kfo9rcm.ga in their email infrastructure. Furthermore, if I conduct a [DNS lookup](https://activedirectorypro.com/use-nslookup-check-dns-records/) to find the mail exchange or 'MX' record for eHarmony.com, it returns this result:

![image](https://user-images.githubusercontent.com/84248865/120120183-4e9f8e00-c161-11eb-9a3b-69126f58353e.png)

You can see here the mail exchanger is eharmony-com.mail.protection.outlook.com. This is the domain for the mail server that eHarmony uses to handle email traffic. Now, we can use this MX record to find its IP address with another DNS query. This time, we will query the 'A' record, which is used to point a domain to an IP address. 

![image](https://user-images.githubusercontent.com/84248865/120120214-7989e200-c161-11eb-8c42-e236f8c57698.png)

This returns two IP addresses. If we use [WhoIs lookup](https://whois.domaintools.com/) to search for these IP addresses, we find that they both point to Microsoft as the organization, which makes sense because the mail exchanger used is Microsoft Outlook. 

Now, if we go back and take another look at the IP address associated with the email I received, it bears no connection to either of the addresses resolved to eHarmony's MX record. So this email could not have come from eHarmony.com.

There are other mechanisms you can check for validity such as [SPF and DKIM](https://woodpecker.co/blog/spf-dkim/), but for this email they were not useful because the sender didn’t actually attempt to spoof the eHarmony.com domain. Rather, they simply named the account "eHarmony Offer" to try to trick the recipient. As you can see in this screenshot, the email already seems very suspicious just by looking at it in Gmail. 

![image](https://user-images.githubusercontent.com/84248865/120082735-34df4780-c08a-11eb-9cfc-9b57a7bf3748.png)

Now, give it a try on your own. See what you can learn by looking at the **Received:** headers of a suspicious email. Just be sure to do it carefully and do not analyze the email directly through the email client, in case of malicious content. For a more in-depth analysis tutorial, check out this [video](https://www.youtube.com/watch?v=nK5QpGSBR8c) by 13Cubed. It's very informative and it really helped me understand the process.

Thank you for reading!

References:

Allen, Robert. “How to Use Nslookup to Check DNS Records.” Active Directory Pro, Active Directory Pro, 17 Feb. 2018, [https://activedirectorypro.com/use-nslookup-check-dns-records/](https://activedirectorypro.com/use-nslookup-check-dns-records/).

Davis, Richard. “Email Header Analysis and Forensic Investigation.” [Https://Www.13cubed.Com/](Https://Www.13cubed.Com/), YouTube, 13 Jan. 2020, [https://www.youtube.com/watch?v=nK5QpGSBR8c](https://www.youtube.com/watch?v=nK5QpGSBR8c).

Dawiskiba, Cathy. “SPF & DKIM for Dummies: What Is It? Why You Want to Set It Up.” Woodpecker Blog, 24 July 2020, [https://woodpecker.co/blog/spf-dkim/](https://woodpecker.co/blog/spf-dkim).

Pressable. “What Are DNS Records? Types and How to Use Them.” Pressable, 11 Oct. 2019, [https://pressable.com/2019/10/11/what-are-dns-records-types-explained-2/](https://pressable.com/2019/10/11/what-are-dns-records-types-explained-2/).

“Whois Lookup, Domain Availability & IP Search - DomainTools.” Whois Lookup, Domain Availability & IP Search - DomainTools, [https://whois.domaintools.com/](https://whois.domaintools.com/). Accessed 16 May 2021.

