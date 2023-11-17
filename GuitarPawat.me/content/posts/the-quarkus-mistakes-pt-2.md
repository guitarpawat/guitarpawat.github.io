---
title: "How to prevent thread blocks in Quarkus"
date: 2023-09-26T21:34:00+07:00
draft: true
toc: true
scrolltotop : true
images:
tags: 
  - java
  - quarkus
  - technical
---
If you got this error...
  await indef
The mistake
  Change warning time
  Use vertx to block
Two types of threads
Handling blocking method
  blocking annotation
Handling blocking and non-blocking in the same endpoint
  MutinyHelper
---
# Why you are getting this error
```text
2566-09-30 23:35:06,011 WARN  [io.ver.cor.imp.BlockedThreadChecker] (vertx-blocked-thread-checker) Thread Thread[vert.x-eventloop-thread-0,5,main] has been blocked for 2006 ms, time limit is 2000 ms: io.vertx.core.VertxException: Thread blocked
	at java.base@17.0.2/java.lang.Thread.sleep(Native Method)
	at org.example.BlockingService.blockingMethod(BlockingService.java:9)
    ...
```
