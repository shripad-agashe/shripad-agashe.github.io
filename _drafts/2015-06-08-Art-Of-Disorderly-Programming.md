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


#Chimera of roll back
One of the interesting things in the solution using two phase commit is visibility of changes. Even though the customer and account has been created it will not be visible outside execution context unless it is committed. This behavior gives a semblance of rollback as well. However consider a scenario when a customer is stored in a database with backing B-Tree and adding a new customer causes the B-Tree to balance. At that point event if the transaction is rolled back, the B-Tree can not be reverted back to its original state. Instead DB engine will compensate for th

#Enter distributed systems

Enter distributed systems and this reasoning is no longer simple. In our case customer and account entities were managed by separate applications. The mode of communication between applications was events in queue. This architectural choice ruled out using distributed transactions or any such form of two phase commit. So we were forced to move away from temporal reasoning i.e. order of actions in system to a solution where the final state is insensitive to order of actions.




#Explain CALM
#Show how Account creation is monotonic activity
#Talk about alternative/When CALM is applicablle
#Take Amazon ordering example i.e. payment


[cs]:http://www.eolss.net/sample-chapters/c15/E1-29-01-00.pdf
