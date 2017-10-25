---
layout: post
title: "Building on quicksand"
date: 2015-12-27 16:25:06 -0700
comments: true
custom_css: Building-on-quicksand

---
##For the want of reliability
This post is inspired by a [paper][bqs] of the same name by Pat Helland. 
The basic idea is: build a reliable system from unreliable components. In this blog post I've attempted to add some of my thinking on this subject apart from what is covered in the original paper. You will be surprised to find that reliability decisions which in turn impact availability have a major impact on system design. Sometimes the connection is visible sometimes it is not.

The idea in the blog post can be nicely summarized as:
>    For want of reliability, redundancy is found.

>    For want of redundancy, asynchrony is found.

>    For want of asynchrony, apology is found.

>    All for the want of reliability.

##Some maths first
Computer systems were not the first complex systems where availability was a key issue. Most of the techniques used for increasing availability have been in use since dawn of industrial age. The techniques mostly rely on probability theory to devise effective strategies to increase availability. So let's get down to some math.

###The product law (or multiplication law)................[1]
This rule states that the probability of simultaneous occurrence of two or more independent events is the product of the probabilities of occurrence of each of these events individually.

###Product law for serial systems
When components of a system are arranged in series then availability of the entire system depends on all the components being available. Hence the system reliability is reliability of all components taken together. 

It can be expressed as:
>   **R<sub>s</sub> = P( A<sub>1</sub> &#x2229; A<sub>2</sub> &#x2229;.... &#x2229; A<sub>n</sub>)**

>where A<sub>1</sub> is probability of availability of system 1, so on and so forth.

>**For example**

>    For a system with 3 components each with 99% availability, 

>    **the system availability** = ( 0.99 * 0.99 * 0.99) = 0.97 i.e. **97%**

###Product law for parallel systems
When components of a system are arranged in parallel, then availability of system depends on at least one of the component being available i.e. system is available even if a single component of the system is available. 

It can be expressed as:
>**R<sub>s</sub> = 1 - P( ( 1 - A<sub>1</sub>) &#x2229; (1 - A<sub>2</sub>) &#x2229;.... &#x2229; (1 - A<sub>n</sub>) )** 

>where A<sub>1</sub> is probability of availability of system 1, so on and so forth.

>**For example**
>
>For a system with 3 components each with 99% availability arranged in parallel, 

>**the system availability** = 1 - ( (1 - 0.99)  * (1 - 0.99)  * (1 - 0.99))
                             = 1 - ( 0.01 * 0.01 * 0.01 )
                            = 0.999999 i.e **99.9999%**



##Box Box Cylinder
As seen in previous section, it is advisable to have components arranged in parallel to improve reliability of systems. Hence architectures and best practices evolved around this idea. Assuming we have a database, these architectures could be referred to as Box-Box-Cylinder architecture. 

![Box Box Cylinder](/images/building-on-quick-sand/B-B-C.jpg)

Fig.1 - BBC Diagram



Couple of best practices that emerged from this architectural style that I would like mention are:

1. Layered Architecture
Layering allows decomposition of responsibilities. It is general technique to reduce complexity at local level by using abstractions. In layered architecture the complex behavior is achieved via different combinations of layers. The Box-Cylinder a.k.a 2-tier architecture is generally extended to n-tiers in modern systems.

2. Stateless layers
State (may be for a single request) at a layer can be stored on either one or all components of the layer. There is no in-between. So if a state is stored on a single component, then the availability of the layer depends on that single component. If the state is stored on all components then it is even worse as the layer's availability to serve a particular request depends on all components being available.This has the effect of making the components serially linked impacting availability characteristics. Hence it is advisable to have the layers stateless.

###But database is still a single node
All though we are able to replicate processing nodes, the database is still a single node and we still need to think about availability. As the layers of system needs to be arranged in series, availability of database determines availability of the system. 

For database layer the strategy is still the same: have redundant components. So it is imperative to replicate state change from one server onto a different server. There are variations of this strategy like real application cluster etc. For the current discussion I would consider most simplistic setup of primary replicating all the changes to secondary node before sending an acknowledgment to client. 

As the client waits for replication to be completed before getting an acknowledgment, the replication adds to the the latency. So in most cases the replication is done off the critical path for response to client. This opens up a window of time where there is a possibility of data loss. The effort is to keep the window of loss of information very small to reduce the impact of loss of information. One example of minimizing the window is transaction abort. Lets consider user issuing writes to DB as part of transaction. Individual write requests are **not** guaranteed to be persisted on primary and on secondary node. The guarantee is now shifted to transaction commit. So effectively w.r.t. time order, the write request by user and guarantee of persistence are delinked. The abstraction for guarantee has moved from an individual write to transaction commit which is a logical(often business) concept. This notion of moving away from system specific concept i.e. read or write from persistent store to a logical concept is a very powerful idea often under utilized in current systems.

##Stretching transaction commit analogy to business logical
As we have seen in previous section, as we move from primitive constructs like storage read/write to transactions, a range of possibilities open up. Of course there are trade offs. We can stretch this further and move level of abstractions from database transactions to business transactions. Since asynchrony enables this stretch, it is important to understand it's impact on design.

###Asynchrony and truth
Pat Helland in his talk "Subjective Consistency" made a very important point that concept of asynchrony is relative to the observer. If we go back to previous example, the writes when persisted are asynchronous to the user who requested them but for the transaction log writer it will be synchronous. Hence for same set of actions, the notion of truth as observed by the user and the log writer may be different at a given point in time. So in any system which employs asynchronous processing technique the system will appear more like in the picture below. 

![Blurred Truth](/images/building-on-quick-sand/async-system-runner.jpg)
[image source:https://www.flickr.com/photos/stevenpisano/16595925953]

In a system with multiple processes, any process will have a fuzzy idea of what is happening around it. You may think that this is applicable for only distributed systems but I can not possible confirm that :-)) (Plug of a dialog from house of cards original uk version in case you missed it). Jokes apart, the scenario is equally applicable in a single system with multiple concurrent threads. [Feral concurrency control paper][bailis-feral-concurrency] is a good read on this subject.

###Appetite for risk for doing thing asynchronously
Fuzziness of truth brings in possibility of wrong assumptions. Wrong assumptions lead to wrong actions and hence a possibility of a loss. Since this is all probabilistic, we hope that things will work out most of the time and sometime we will have to bear the consequences of our choices. The quantum of loss we are ready to bear will decide how much fuzziness we are ready to live with. The level of fuzziness represents trade-off between benefits either in terms of availability, latency vs quantum of possible loss. Hence the decision shifts from being a technical decision to a business model decision. This factor alone makes lot of developers hesitant and the project managers even more. 

###Memories, Guesses and Apologies
All we have discussed so far can be nicely summarized in these three words. It gives us a nice vocabulary to communicate these complicated ideas.

From theories of physics we know that there is no concept of "now". Justin  Sheehy has written an excellent [article][js-now] on this. Any process observing external entities is likely be looking at version of the entities at a point in time. The processing unit generally takes a bet that the state of external world is still the same as what it is seeing and proceeds with enforcing business rules. Some times the bet can go wrong and the system will enforce incorrect rule. Hence the system should be ready to apologize and compensate for the wrong enforcement of the business rule. The compensation could be reverting the local processing or doing additional work to be consistent with rules based on current knowledge etc.

One import point to note is that even if a software system perfectly represents the business state, it is still physically disconnected from real world. Events like accidents, floods or any other natural calamity will require the system to take corrective actions. So if we treat apology mechanism as a safety net, we can stretch the systems to work on partial knowledge. Typically this is done to increase availability, reduce latency or work in off line mode. These choices typically result in additional revenue as against the cost of being incorrect sometimes.

 Pat Helland has nicely summarized this idea. In Pat's words:
>Arguably, all computing really falls into three categories: memories, guesses, and apologies.The idea is that everything is done locally with a subset of the global knowledge. You know what you know when an action is performed. Since you have only a subset of the knowledge, your actions are really only guesses. When your knowledge as a replica increases, you may have an “Oh, crap!” moment. Reconciling your actions (as a replica) with the actions of an evil-twin of yours may result in recognition that there’s a mess to clean up. That may involve apologizing for your behavior (or the behavior of a replica).


### More on apologizing
First step in any apology process is to find out if we need to apologize at all. That means we need a way to find out the promises that could not be fulfilled. An individual process or system can tell truth from its own viewpoint. However to verify any claim of truthfulness we will need a third party frame of reference. Typically it is provided by reconciliation process. The reconciliation process verifies what was asked and what was done. Since it provides a third party frame of reference, failure modes of communication mechanisms like failing queues, network connections can also be handled gracefully. Reconciliation process provides a great deal of flexibility. It can be run at different intervals based on severity of thing it is trying to verify. It can can correct any anomaly it finds based on set of rules or route it for human intervention. 

###Need for Unique Id
Often, time which is nothing but monotonically increasing tick helps in reasoning what has happened in what sequence. However in asynchronous processing, time ticks does not help much for reasoning. We need to create additional synthetic ids to track progress of a business process. There are number of schemes available for managing Ids. Most common scheme is to have a set of three Ids: First id indicates ongoing transaction, second id indicates parent transaction and third id indicates the current transaction. Following this scheme we can easily build DAG of the business process.


###Repercussions of apology process
Apology process provides a safety net using which we can stretch system capabilities. Metaphorically they provide the brakes for business processes. In my experiences with software systems utilizing asynchronous processing: apology process with a third party frame of reference solves quite a few problems which otherwise would have required complicated solutions.

Apology oriented mindset has significant impact on design thinking of developers. In this model, verification of correctness happens after the fact. Hence the design tends to follow more of choreography approach rather than orchestration approach. This naturally leads us to  of micro services and domain driven design.

##Thats all folks
We started from reliability aspect of systems. The reliability forces redundancy on us hence we move into distributed components. Distributed components tend to have latency concerns hence we move to asynchronous model. Asynchronous model relies on fuzzy knowledge of the world hence we need apology process. So I am going to say sorry now and move on.... 


[bqs]:http://db.cs.berkeley.edu/cs286/papers/quicksand-cidr2009.pdf
[mga-helland]:http://blogs.msdn.com/b/pathelland/archive/2007/05/15/memories-guesses-and-apologies.aspx
[js-now]:https://queue.acm.org/detail.cfm?id=2745385
[bailis-feral-concurrency]:http://blog.acolyer.org/2015/09/04/feral-concurrency-control-an-empirical-investigation-of-modern-application-integrity/
