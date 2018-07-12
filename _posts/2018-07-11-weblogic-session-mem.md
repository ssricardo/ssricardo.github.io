---
layout: page
title:  "Weblogic Session Memory Analysis"
date:   2018-07-11 00:00:00 -0300
categories: java weblogic memory session heap analysis
---

> **QUICK TIP** about memory analysis of the Web Session on Weblogic.

This article is a follow up of *Java Memory Analysis with Eclipse MAT*. So, some concepts shown there are used here.  
Therefore, you should take a look on the previous article, if you need.

Here it's shown how to find the data from Web Session in a **heap dump** from an instance of **Weblogic** server.  

> TLDR: Look up the classes *MemorySessionData* and / or *AttributeWrapper*

## Use

This kind of analysis is useful in the following situations in a Weblogic server:  

* You are getting OutOfMemoryError 
* You are struggling with any memory leak 
* You have (or foresee) problems while scaling your application  

And other similar situations...  

## Setup

In this task, we need:

* To get a *heap dump* from the target server (check the previous article, if needed), a
* An application for *heap* analysis. In this case, we are using **Eclipse MAT**

## Action!

Open up the *heap* on MAT.  Using either *Dominator tree* or *Histogram*, the class you look up is:

> `weblogic.servlet.internal.session.MemorySessionData`

Or just "MemorySessionData".  
This represents user web sessions. Now, it's possible to check the class's number of instances (objects), as well as its total size.  

The next step is to check the instances list itself.  
  
If you are using the *Dominator tree*, then objects are listed already.  
Otherwise, when using *Histogram*, then it's necessary to list the objects:  

> -> Right click on the class item -> List Objects -> with incoming references

Now, pick one of them to be analyzed.  Generally, a good candidate is one of the biggest instances (It's likely that the problem is there).  

To **detail** a given instance:  

> -> Right click on the item -> List objects -> with outgoing references

Inspect this object. A proposed approach is to select the object with highest *Retained Heap* and to expand it. 
Do it successively according to your needs.  
The real attributes are wrapped into:  

> `weblogic.servlet.internal.AttributeWrapper`

So, you so might want to use it to filter results some time. 

*!EOF*

---