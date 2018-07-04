---
layout: page
title:  "Java Memory Analysis with Eclipse MAT"
date:   2018-07-04 00:00:00 -0300
categories: java memory eclipse mat analysis heap
---

> DRAFT: I'll continue this article soon...

> This articles show an introduction to memory analysis in Java applications. It shows an overview of heap dump and the Eclipse MAT tool.

## Java memory analysis

Throughout applications lifecycle sometimes we face problems like memory leak, or even without a manifested error, we may try to improve some non-functional feature. 

Java platform provides some ways of analysing application's memory. Coming up with one way of categorize them:  

* Real time analysis
* Static analysis

In **Real time analysis** it's possible to introspect the memory as it is while the application is running. This may be achieved using **JMX** which is the API for *management*. Another way is through **profiling** tools like *jprofiler* or *Appdynamics* (a great one).

**Static analysis** is done through a **snapshot** of Java heap in some moment. This is shown in this article.

## Heap dump

*Heap dump* refers to the action of creating the snapshot of Java heap. This process results in a binary file, which may be very big, depending on the target process's memory. 
Getting that, is simple! There are a few ways:

**jmap**: 

One of the simplest and most used ways:

> jmap -dump:format=b,file=my_dump.hprof 78965

In this case, *78965* indicated the process ID in the OS. 

**jvisualvm**:

In this way, it's possible to do it using a GUI application. You need to have access to this process using *JMX* through.  

1. Run the application (`.{your jdk bin directory}/jvisualvm`)
2. On the left pane, select the process
3. On the right pane:
    1. Monitor
    2. Heap Dump
4. On the left pane, click on the created entry, and "Save as..."

**jcmd**:

From Java 7 on, there is another tool. It's said that this tool, it is faster and created dumps are smaller. (I haven't checked it out)  

> jcmd 98765 GC.heap_dump /home/my_dump

Other ways of doing it are shown in [this article](https://dzone.com/articles/how-to-capture-java-heap-dumps-7-options), including one triggered automatically when an OutOfMemory occurs.

### Working with the dump

Ok, now we have the data... so, how to use it?!

## Eclipse MAT

Eclipse MAT is a free / open source tool based on the well-known Eclipse (oh, really?!) targeted to Java heap dump analysis. MAT stands for Memory Analyzer Tool. 

Check out the [project's site](https://www.eclipse.org/mat/)

It provides a visual environment to navigate / explore the given Java heap snapshot.  
Moreover, it has tools that tries to figure out memory leak or possible problems automatically.