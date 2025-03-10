---
title: "The Myth of the 13 DNS Root Server Addresses"
description: "One of the most common internet myths is that there are only 13 *physical* DNS root servers worldwide"
date: 2025-03-10
weight: 2
categories: ["DNS", "RootServers","13RootServers"]
tags: ["DNS", "RootServers", "Networking", "Anycast", "Tech","Networking101"]
draft: false
---

# Why Are There Only 13 DNS Root Server Addresses?


## Introduction
If you’re reading this blog post, chances are you’re already reaping the benefits of a highly distributed system for resolving domain names. Every time you type in a website address or click a link, the Domain Name System (DNS) springs into action, translating your friendly “www” addresses into numerical IP addresses. But you might have come across a puzzling fact: **there are only 13 DNS root server addresses.** Let’s explore why that is and bust a common misconception!

> **“Why are there only 13 DNS root server addresses?  
> A common misconception is that there are only 13 root servers in the world. In reality there are many more, but still only 13 IP addresses used to query the different root server networks. Limitations in the original architecture of DNS require there to be a maximum of 13 server addresses in the root zone. In the early days of the Internet, there was only one server for each of the 13 IP addresses, most of which were located in the United States.  
>  
> Today each of the 13 IP addresses has several servers, which use Anycast routing to distribute requests based on load and proximity. Right now there are over 600 different DNS root servers distributed across every populated continent on earth.”**

## The Myth of the 13 Servers
One of the most common internet myths is that there are only 13 *physical* DNS root servers worldwide. Imagine if that were true—nearly the entire planet’s DNS lookups would be handled by a mere handful of machines! That could be a bit scary, like having only 13 vending machines for coffee for everyone on Earth. (We’d never get caffeinated enough!)

In truth, the number 13 corresponds to **13 unique IP addresses**, not 13 actual physical servers. 

## The Historical Reason
When DNS was first developed, its architecture was limited in how many name server addresses could be listed in the root zone. The engineers decided on a maximum of 13, due to technical constraints related to:

1. **Protocol Limitations:** Early DNS packets had size limitations, affecting how many root server entries could be included.
2. **Network Efficiency:** DNS was originally designed for a smaller internet, not the mega-network we use today.

## Anycast Magic
Fast-forward to the modern era, and we have **Anycast routing** to save the day. The idea behind Anycast is delightfully simple yet highly effective:

- **Multiple Servers, One IP**: You have multiple servers around the globe, but each shares the *same* IP address.
- **Geographical Proximity**: Internet traffic is routed automatically to the nearest or least busy server using this shared IP.
- **Load Balancing**: The load is spread among many servers, increasing reliability and speed.

Thanks to Anycast, each of those 13 “root server addresses” can represent a cluster of physical servers scattered across multiple continents. As of now, there are **over 600** physically distinct servers operating under those 13 addresses, ensuring global coverage and robust DNS resolution.

## Why This Matters
- **Resilience**: With so many distributed servers, DNS remains stable even if some servers go down.
- **Speed**: You’re usually routed to the nearest root server, which means faster website resolutions.
- **Scalability**: More servers can always be added to the clusters under the same IP to handle increased global internet usage.

## Fun Fact
You might stumble upon root servers named with letters—like `A`, `B`, `C`, etc. These labels correspond to each of the 13 IP addresses. For example, “A” is one of the addresses (named `a.root-servers.net`), and it has multiple physical servers worldwide.

## Final Thoughts
So, the next time you hear someone say there are only 13 DNS root servers, feel free to put on your DNS superhero cape and explain the real story. **The “13” refers to IP addresses, not the number of physical machines!** In reality, these IP addresses direct you to hundreds of actual servers via the Anycast wonder.

If you’re ever curious about where your nearest root server is located, there are DNS tools out there to show your DNS route. It’s a neat exercise in seeing how globally connected the internet really is—even behind the scenes.


---