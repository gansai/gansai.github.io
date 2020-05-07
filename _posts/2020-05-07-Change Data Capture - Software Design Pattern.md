---
layout: post
title: Change Data Capture - Software Design Pattern 
published: true
disqus_identifier: Change Data Capture - Software Design Pattern 
comments: true
---

# Change Data Capture - Software Design Pattern

Change Data Capture or CDC is an approach to data integration, generally used in enterprise applications. 

### What is Change Data Capture ?

There are 3 things which happens within CDC products:

1. Identification of data
2. Capture of changes
3. Delivery of changes



**Debezium** is one of the CDC products, which is 

- open-source
- distributed platform

In this post, I wanted to take a new approach to writing blog posts. Code commentaries.

Basically there are 2 motivations:

1. I want to read more code and understand code
2. This is something I have not done yet, in my blogging efforts.

Code commentary:

Language: Java

Repository: Debezium

Package: package io.debezium.pipeline;

Class: ChangeEventSourceCoordinator

Github Path: https://github.com/debezium/debezium/blob/master/debezium-core/src/main/java/io/debezium/pipeline/ChangeEventSourceCoordinator.java

{% gist anonymous/07fa1df33c5c450362175eb11e462163 %}

### Commentary:

1. At line: 41, *@ThreadSafe annotation* , with this annotation. This annotation generally captures design intent. Basically this annotation is used to document thread safety, denoting that, this class: ```ChangeEventSourceCoordinator``` is thread-safe.  You can check, that one of the recommendations laid out by [SEI CERT Oracle Coding Standard for Java](https://wiki.sei.cmu.edu/confluence/display/java/CON52-J.+Document+thread-safety+and+use+annotations+where+applicable), is to document thread safety. This can be added to our kitty of best practices for coding.

2. At line: 46, Usage of Duration class: 

   1. ```java
      private static final Duration SHUTDOWN_WAIT_TIMEOUT = Duration.ofSeconds(90);
      ```

      Duration class was introduced as part of Java 8. This is later used within stop() method, as a parameter to ```awaitTermination()``` for shutting down executor. Generally the old way of coding is to use a ```Long``` to have some value of milliseconds. But, using Duration is a cleaner approach and improves readability too.

3. Then, we can see the organization of import statements. 

   1. Firstly, imports related to java jdk
   2. Secondly, imports to external 3rd party, for example: sl4j for logger component, kafka for kafka-connect component
   3. Thirdly, imports from own component ( from self )