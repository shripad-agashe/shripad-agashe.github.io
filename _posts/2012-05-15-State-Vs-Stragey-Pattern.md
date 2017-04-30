---
layout: post
title: "State Vs Strategy Pattern"
date: 2012-05-15 16:25:06 -0700
comments: true

---

I was having a casual discussion with an ex colleague.  The topic went on to patterns discussion and finally we ended at usual discussion about state vs strategy pattern. All though the discussion did not end conclusively, it remained on my mind for a while culminating into this blog post.

Typically State/Strategy pattern will be used where the behavior is conditional i.e. conditional statements will be replaced with objects with specific behavior. The collaboration of participants is what sets strategy and state apart.  I've tried to mark the differences between them on various parameters.

##Participant Coupling
In state pattern, participants (at least the concrete implementations & context) have knowledge of each other. In most cases context is going to manage the state transition, so it has intimate knowledge of the implementations and their responsibilities. What it means is that context is aware of the capabilities of the concrete implementations it needs.
In strategy, context will not know of the objects it has composed. The context will refer concrete implementations through its abstraction. Hence context will never be able to configure the strategies or change strategies. It means, the context is not aware of capabilities of concrete implementations. 

##Configuration of Implementation objects in Context

In state pattern, the client typically will just use the context out of the box i.e. it will not be able to configure the context with states at different times.  The only way for the client to change the composition of context is thru state mutation of the context. The context or concrete state implementations will handle transition based on the behavior required for the state after mutation.

In strategy, since context refers to concrete implementations thru abstraction, it can not take responsibility for managing lifecycle of the concrete implementations. The client acts as orchestrator to configure concrete strategy in context objects.This may force context to use services from strategy abstraction in idempotent manner since it can reason only about abstraction and not concrete implementations. 


##Refactoring

IMO the participant coupling described above has an effect on refactoring abilities. Since state pattern has close object group i.e. context composition is hidden from clients, its clients typically would use it in a uniform way vis-a-vis strategy pattern. In strategy, clients will configure context with strategies hence, it is likely to have varied usages in terms of different strategies used by different clients.
This will make state pattern easier to refactor as compared to strategy pattern.

I hope some the points mentioned in post help in clarifying the differences in state and strategy pattern and help in making right design decisions.
