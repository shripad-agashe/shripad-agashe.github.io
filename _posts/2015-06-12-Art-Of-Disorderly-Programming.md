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

For creating an account, a customer can be an existing customer or a new customer. The scenario we were trying to solve is creating an account for a new customer.


#Simple Solution
In a non distributed system supporting 2 phase commit functionality, the solution is simple. In a single session, first create a customer then the account with customer created earlier. This is depicted in sequence diagram below:

![Alt text](/images/create-account-seq.jpg)

Reasoning about this solution is simpler. If account creation fails, customer is not created as well. At any point of time account can not be created without a customer and creation of account and customer is visible only to current execution context. The semantics and implementation of this approach is well understood as this is the primary design choice for most developers.

One of the interesting things in the solution using two phase commit is visibility of changes. Even though customer and account has been created it will not be visible outside execution context unless it is committed (assuming that appropriate transaction isolation levels have been set). The transaction coordinator will handle commit/rollback ensuring entities in valid state is visible to external world. __[Tangent]__ Rollback may just be an illusion a explained by Pat Helland in Open Nested Transactions.


 All though two phase commit handles this via session handling, it can very well be modeled via business statues i.e. created-but-not-valid, created-and-valid, deleted etc. Business statuses are fairly common. Mostly they are used to mark deletion as logical deletion. This technique is very useful technique and we will use in our solution. 

#Enter distributed systems

Enter distributed systems and this reasoning is no longer simple. In our case customer and account entities were managed by separate applications. The mode of communication between applications was events in queue. This architectural choice ruled out using distributed transactions or any such form of two phase commit. So we were forced to move away from temporal reasoning i.e. order of actions in system to a solution where the final state is insensitive to order of actions.


#Determining if eventual consistency is even applicable
There are efforts to figure out a way to ensure that programs are eventually consistent by design in face of temporal non determinism. One such approach is CALM. CALM stands for consistency as logical monotonicity. [CALM][calm] specifies when an eventual consistency paradigm is applicable and which operations can guarantee consistency in presence of temporal non determinism. From what I understood, operations which can be modeled as sets and expressed as selection, projection and join can be implemented as eventually consistent i.e. these programs have increasing consistency as more input data is received.  These sort of operations can be termed monotonically increasing consistent operations. 

Operations such as aggregation or negation are either data dependent or order sensitive can only be implemented via blocking operation. 

#TODO - Account as CALM
#TODO - Final Solution Schematic




[cs]:http://www.eolss.net/sample-chapters/c15/E1-29-01-00.pdf
[calm]:http://db.cs.berkeley.edu/jmh/calm-cidr-short.pdf
[ec-bailis]:https://queue.acm.org/detail.cfm?id=2462076