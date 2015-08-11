---
layout: post
title: "Art of Disorderly Programming"
date: 2015-06-08 16:25:06 -0700
comments: true
categories: {"ACID", "Consistency", "DDD", "Order of events"}
---
#Do we have to decide is it chicken or egg?
Why can't we let them come in any order and decide the outcome as cumulative effect of two events? Well I've stretched the analogy quite a bit. But wouldn't be wonderful if it can be achieved?  
Often we tend to reason in terms of temporal order of events and get fixated on the order than the eventual outcome. In a single system, ensuring temporal ordered ensured correct eventual outcome. However in partitioned (or distributed) systems temporal order is very hard to enforce. As a corollary from point of view of single system, ensuring correct outcome become a challenge in distributed systems. In my current project, there was a scenario which lent itself to similar treatment. This blog is my thoughts on the subject and hopefully it will help you ( as well as me).

*__[Tangential]__ If we compare this with what has been studied in area of __[complex system][cs]__, we will find a rich trove of information and solutions to deals with some of these problems.*

#Scenario
Consider a bank account. Let's say there is a constraint that valid account can  exists only if it has a valid customer associated with it. In a simplest way, it can be represented as shown below:

![Alt text](/images/account-class-diagram.jpg)

For creating a account, a customer can be an existing customer or a new customer. The particular scenario we were trying to solve is creating an account for a new customer.

The system we are working on is loosely coupled. Customer domain is managed separately from account domain. Thinking from temporal perspective, typical business process rule is: ensure customer exists and then create an account for that customer. 


![Alt text](/images/create-account-seq.jpg)



#Naive Solution
In 2PC world it is relatively simple to 

#Why it is important

#Explain CALM
#Show how Account creation is monotonic activity
#Talk about alternative/When CALM is applicablle
#Take Amazon ordering example i.e. payment





[cb]:http://martinfowler.com/bliki/CircuitBreaker.html
[ri]:http://www.amazon.com/gp/product/0978739213?ie=UTF8&tag=martinfowlerc-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0978739213
[lz]:http://homes.cs.washington.edu/~lazowska/qsp/Images/Chap_03.pdf
[wiki-little-law]:http://en.wikipedia.org/wiki/Little's_law
[cs]:http://www.eolss.net/sample-chapters/c15/E1-29-01-00.pdf
