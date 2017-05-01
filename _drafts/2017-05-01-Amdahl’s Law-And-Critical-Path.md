---
layout: post
title: "Amdahl’s law and changes in critical path"
date: 2017-04-30 16:25:06 -0700
comments: true

---
Augmentation of hardware and software resources is a classic technique that is often adopted by IT teams to improve the performance of applications. However, it is commonly observed that, contrary to expectations, the increase in the computing resources does not necessarily lead to improvement in the performance, as expected. This situation presents unique challenges to the IT teams as they grapple to analyze the reason for the inefficacy of the technique in improving performance.

Failure to enhance performance can be attributed to multiple causes - some trivial like incorrect configuration of the new environment as well as some severe like architectural flaws or mandatory architectural requirements that prevent scaling. One such aspect of the architecture that can have a significant bearing on the performance improvement is the behavior of the critical path in the application flow under varying workload conditions. The path termed as Critical Path is the longest path (in terms of time) in the system and determines the minimum time for completion of requests. Analysis of the Critical Path in the application is a vital check point as it can expose reasons for failure in achieving expected performance improvements. Normally, it is assumed that the critical path in the system does not change with changes in the workload. However, in applications which have parallel application flows, this assumption may lead to incorrect performance analysis thus leading to solutions which do not address the actual performance bottleneck.

In subsequent sections, I will explore a methodology for identifying the critical path under different workloads and its implication on the performance analysis process.

##Performance Issues - A typical solution
A real-life example of an enterprise application that has multiple parallel application flows is presented below. The example illustrates a Billing Mediation System that calculates the billing amount and then routes the bills to appropriate Billing servers.

An example is the Billing Mediation System diagramed below. The BMS calculates the billing amount and, then, routes the completed bill to the appropriate Billing servers.

![Billing system](/images/amdahls-law/billing-system.jpg)

The Billing Gateway component in the mediation system receives the requests and then invokes the Bill Calculator and the Bill Router components, in parallel, to reduce the time taken to processes the request. The generated bill is passed to the Bill Dispatch component which dispatches the bill to the appropriate Billing server as identified by the Bill Router. It is important to note that the bill cannot be dispatched unless both Bill Calculator and Bill Router complete their processing. The Bill Router component requires significantly more processing; hence, the service time of the request in the Bill Router is higher than the Bill Calculator. The Bill Router component becomes the critical path.

Performance tests were conducted at low loads in a scaled down version of the production environment. Performance models [1] were created and parameterized with measurements (CPU, IO, Response time) taken during testing and the performance characteristics were extrapolated to understand the potential performance characteristics at the expected load. Hypothesis based on the performance models indicated that the application would meet the desired performance goals once deployed in the target production environment, which has considerably more resources. The application was deployed in the production environment and tested at the actual workload. It was observed that the actual performance characteristics did not meet the predicted performance characteristics. Discrepancies were not observed in the environment settings and none of the resources on the system (CPU, memory, network, etc.) were saturated.

##INITIAL ANALYSIS & OBSERVATIONS

The Billing Mediation System can be envisaged as a network of interdependent activities; hence the critical path technique [2] was adopted when analyzing the performance of the application. The Critical Path technique was used to identify the longest path in the network as it determines the response time of the request. Observation of component service time in the production environment revealed that the Bill Calculator component consumed more time for completion than the Bill Router component, contrary to observations in the performance testing environment. Analysis of the component service time indicated that the critical path observed in the production environment (Billing Gateway- Bill Calculator - Bill Dispatch) was different than the path observed in the performance test environment (Billing Gateway- Bill Router- Bill Dispatch). The application team had assumed that the critical path will remain unchanged across workloads and hence the response time figures from the test environment were extrapolated to derive the response time/ throughput for a given workload.

In the subsequent section we try to validate the assumption that the critical path remains unchanged across workloads. This will lead us to correct identification of critical path for any given workload.

###PERFORMANCE FORECASTING AND CRITICAL PATH ANALYSIS
Amdahl’s Law gives maximum speedup possible in parallel processing systems. Amdahl's law states that if P is the proportion of a program that can be made parallel, i.e. benefit from parallelization, and (1 − P) is the proportion that cannot be parallelized, i.e., remains serial, then the maximum speedup that can be achieved by using N processing threads is:

![Billing system] (/images/amdahls-law/amdahls-law.jpg)



