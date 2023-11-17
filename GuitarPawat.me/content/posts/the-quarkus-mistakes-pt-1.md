---
title: "Why your Quarkus reactive codes are not executing?"
date: 2023-09-18T00:13:00+07:00
draft: false
toc: true
scrolltotop : true
images:
tags: 
  - java
  - quarkus
  - technical
---
Smallrye Mutiny is essential for creating your reactive application in Quarkus.

People new to Quarkus and reactive programming may be confused with Smallrye Mutiny concepts and make many mistakes in the code.

This is one of the mistakes I found many people made while they were new to Quarkus.

In this post, I will focus on Mutiny's Uni rather than Multi since it is simpler for the code.

## Uni will not be executed if there is no subscriber

Suppose you have an endpoint that is used to start some background tasks and just returns the text that the server received the task and is being started.

```java
package org.example;

import io.smallrye.mutiny.Uni;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.Produces;
import jakarta.ws.rs.core.MediaType;
import org.jboss.resteasy.reactive.RestQuery;

@Path("/hello")
public class GreetingResource {

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello(@RestQuery String taskId) {
        startBackgroundTask(taskId);
        // Returns that the server received the request
        return "Processing " + taskId;
    }
    
    Uni<Void> startBackgroundTask(String taskId) {
        return Uni.createFrom()
            .voidItem() /* Should be some logic in real world */
            .onItem().delayIt().by(Duration.ofSeconds(5))
            .invoke(() -> System.out.println("executing background task for id: " + taskId));
    }
}
```

After you call `GET /hello` with `taskId = testId`, you will the response as `Processing testId`.

But there is no message in the console. Why?

Because Uni executes the task ***lazily***. To make Uni execute the code, you must subscribe it to something.

There are two main ways to subscribe to Uni:

## Subscribe to Uni as Quarkus Endpoint Response

You can make Quarkus subscribe to your task as the response of the endpoint by returning the Uni from the controller method.

```java
@GET
@Produces(MediaType.TEXT_PLAIN)
public Uni<String> hello(@RestQuery String taskId) {
    return startBackgroundTask(taskId)
        .replaceWith(() -> "Processing " + taskId);
}
```

Now, it will print `executing background task for id: testId` to your console.

But there is one drawback:

The response will only return after the background task is finished.

If it is a long task, the user will have to wait for a long time.

## Subscribe to Uni with Callback

To fix the problem from above, you need to subscribe to something else.

It could be some callback method or just an empty callback like in the code below.

```java
@GET
@Produces(MediaType.TEXT_PLAIN)
public String hello(@RestQuery String taskId) {
    startBackgroundTask(taskId)
        .subscribe().with(x -> { /* Callback logic */ });
    return "Processing " + taskId;
}
```

In this example, the response will be returned immediately.

The background task will be handled in another thread and will call the callback after it is finished.

# Conclusion

There are two ways to make your Uni get executed depending on how you handle the return value of Uni:

1. If the return value of Uni needs to return to the user: Subscribe with the Quarkus controller method.
    
2. If the return value of Uni does not need to return to the user: Subscribe with callbacks.