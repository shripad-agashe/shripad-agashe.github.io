---
layout: post
title: "Circuit breaker is about slow response"
date: 2014-11-25 16:25:06 -0700
comments: true
categories: {"performance", "latency", "soa"}
---


In his seminal book [Release It!][ri] Michael Nygard elicited the "Circuit Breaker" pattern. He documented the pattern as one of the fundamental stability pattern. However he did not give any implementation schematic and also the documentation does not go in depth about the logic and fundamental need for this pattern. As mentioned in the book, circuit breaker pattern is for preventing cascading failures, unbalanced capacities, slow responses. But most often, it is coupled only with timeout tracking. In his blog about [Circuit Breaker][cb] Martin Fowler also referred this pattern only for tracking failures. I think using circuit breakers only in case of failures is inappropriate at least from a logical perspective if not implementation perspective.

#Response time and timeout

In some sense, timeouts can be viewed a class of erroneous responses. With this view point, timeout is a case of extremely slow response. If the remote service starts returning responses with latency close to timeout, circuit breakers which track timeout errors would be unable to guard against remote services. The effect of slow response is almost similar to a timeout case. In both cases, the effect is increase in concurrent number of system resources being used from normal assumed level. This behavior can be explained easily with [Little's law.][wiki-little-law]. More over using Little's law for this analysis provides insight into need for circuit breaker in the first place.

#Little's Law and its repercussion

In queuing theory any resource be it human or machine is considered a service center or consumer. Hence it is an excellent tool to model users of application, application and hardware in combination.

Littles law is presented as:
<br>

**N = X * R** 
<P>
	where<br>
		N = Number of users in the system typically known as concurrency<br>
		X = Throughput – Number of completed discreet events per unit time<br>
		R = Response time – Time taken for completion of discreet event under observation
</p>

The law can be explained further in relation to figure below. <br><br>	
![Alt text](/images/circuit-breaker-seq.jpg)

<br>Here, client invokes B and C which are remote services. 
<br>Let For service **B**, **response time** be ***B<sub>1</sub>*** and **thoughput be X<sub>1<sub>**
<br>Let For service **C**, **response time** be ***C<sub>1</sub>*** and **thoughput be X<sub>1<sub>**

<br>Now the overall response time for A = **Processing time at A** + **B1** + **C1**

So, by Little's law **N<sub>A</sub>** = ( **Processing time at A** + **B1** + **C1** ) * **X<sub>A</sub>**
<br> Hence **N<sub>A</sub>** &prop; **B1** as well as **N<sub>A</sub>** &prop; **C1** 

Generally service response timeout value is set at much higher level than average response time value typically 4 to 10 times. Hence in case of persistent timeout response or near timeout response from remote service would mean 4 to 10 times increase in concurrency at client. Circuit breakers can alleviate this 

#Design considerations









[cb]:http://martinfowler.com/bliki/CircuitBreaker.html
[ri]:http://www.amazon.com/gp/product/0978739213?ie=UTF8&tag=martinfowlerc-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0978739213
[lz]:http://homes.cs.washington.edu/~lazowska/qsp/Images/Chap_03.pdf
[wiki-little-law]:http://en.wikipedia.org/wiki/Little's_law
