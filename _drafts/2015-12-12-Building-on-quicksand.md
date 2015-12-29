---
layout: post
title: "Building on quicksand"
date: 2015-12-12 16:25:06 -0700
comments: true
categories: {"performance", "latency", "soa"}
---
This post is inspired by a [paper][bqs] of the same name by Pat Helland. 
The basic idea is around how do we build reliable system from unreliable components. In this blog post I've attempted to add some of my thinking on this subject apart from what is covered in the original paper. The content mostly centers around availability. I've tried to cover ways of achieving high availability and it's impact on design choices.

##Get ready for some math
Computer systems were not the first complex systems where availability was a key issue. Most of the techniques used for increasing availability have been in use since dawn of industrial age. The techniques mostly rely on probability theory to devise effective strategies to increase availability. So let's get down to some math.

###The product rule (or multiplication law)................[1]
This rule states that the probability of simultaneous occurrence of two or more independent events is the product of the probabilities of occurrence of each of these events individually.

###Product law of reliability
When components of a system are arranged in series then availability of entire system depends on all the components being available. Hence the system reliability is reliability of all components taken together. 

It can be expressed as:
    R<sub>s</sub> = P( A<sub>1</sub> &#x2229; A<sub>2</sub> &#x2229;.... &#x2229; A<sub>n</sub>) .........from[1]

**For example**
    For a system with 3 components each with 99% availability, 
    the system availability = ( 0.99 * 0.99 * 0.99) = 0.97 i.e. **97%**

###Product law of unreliability 
When components of a system are arranged in parallel, then availability of system depends on at least one of the component being available i.e. system is unavailable if all the components of the system are unavailable. 

It can be expressed as:
R<sub>s</sub> = 1 - P( ( 1 - A<sub>1</sub>) &#x2229; (1 - A<sub>2</sub>) &#x2229;.... &#x2229; (1 - A<sub>n</sub>) )

**For example**
    For a system with 3 components each with 99% availability, 
    the system availability = 1 - ( (1 - 0.99)  * (1 - 0.99)  * (1 - 0.99))
                            = 1 - ( 0.01 * 0.01 * 0.01 )
                            = 0.999999 i.e **99.9999%**



##Box Box Cylinder
As seen above it is clearly advisable to have components in parallel to improve reliability of systems. Hence architectures and best practices evolved around this idea. Assuming we have a DB, these architectures could be referred to as Box-Box-Cylinder architecture. 

![Box Box Cylinder](/images/building-on-quick-sand/B-B-C.jpg)

Fig. - BBC Diagram



Couple of best practices that emerged from this architectural style that I would like mention are:
###Layered Architecture

###Stateless layers
State (may be for a single request) at a layer can be stored on either one or all components of the layer. There is no in-between. So if a state is stored on a single component, then the availability of the layer depends on that single component. If the state is stored on all components then it is even worse as the layer's availability to serve a particular request depends on all components being available.This has the effect of making the components serially linked impacting availability characteristics. 

###DB is still a single node
All though we are able to replicate processing nodes, the DB is still a single node at least logically and sometimes physically as well. But we still need to think about availability. Hence for DBs especially availability  strategies adopted can be broadly categorized as either Active-Active or Active-Passive. 
### Active-Active Strategy
Fig 1. Explains this strategy. The idea is to replicate state change on one server to a different server synchronously. 








[bqs]:http://db.cs.berkeley.edu/cs286/papers/quicksand-cidr2009.pdf
