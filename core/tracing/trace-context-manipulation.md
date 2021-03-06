---
title: Kamon | Core | Documentation
layout: documentation
---

TraceContext Manipulation
=========================


Starting a Trace
----------------

The `TraceRecorder` companion object provides a simple API to create, manipulate and finish a `TraceContext`. To start a
new trace use the `TraceRecorder.withNewTraceContext(..)` method providing a name and optionally a trace token as show
here:

```scala
TraceRecorder.withNewTraceContext("sample-trace") {
  // All code in this block has the same TraceContext.

}
```

It is really important to understand that the TraceContext will **only** be available during the execution of the
specified code block. Any event generated from outside that block will either belong to higher level trace in case of
nesting or to no trace at all. Ideally, the code that initiates a request in your application should be wrapped in one
of these blocks, then Kamon will take care of propagating the TraceContext to all related events and finally, a call to
`TraceRecoerder.finish(..)` closes the trace.

Even while it is important to understand how to start and finish a trace, in the common use case you are building your
application using one of the supported toolkits, Spray or Play!, that means that you don't need to manage the trace
yourself, Kamon already knows when a request starts and finishes within these tools.


Propagation through Actor Messages
----------------------------------

### Tell, ! and Forward ###

If a TraceContext is available when sending a message to an actor, Kamon will capture that TraceContext and make it
available when processing the message in receiving actor. This remains true regardless of whether your are doing a
regular tell, using the `!` operator or forwarding a message to a ActorRef.

```scala
TraceRecorder.withNewTraceContext("sample-trace") {
  actor ! "Some Message"
  actor.tell("Some message", sender)
  actor.forward("Forwarded Message")
}
```

In this particular case, the three messages will propagate the same TraceContext, since they were originated from the
same block of code.

### Ask, ? ###

When you send a message using the ask pattern the TraceContext is also propagated, but additionally the TraceContext is
also available when executing the returned future's body and any registered callbacks:

```scala
val responseFuture = TraceRecorder.withNewTraceContext("sample-trace") {
  actor ? "Ask Message"
}.mapTo[String]

responseFuture.map { response =>
  /* The same TraceContext available when asking the actor is available when executing this callback. */

}
```

### Pipe Pattern ###

When using the pipe pattern, the TraceContext available when the pipe call was made (not when completing the future) is
also made available when processing the message in the target actor.

### Supervision Messages ###

When one of your actor fails it is the responsibility of its parent to determine what action to take based on the
provided supervision strategy. The failure notification as well as the delivery of the supervision directive being to
apply happens through system messages that are not directly visible to users and do not use to the same mailbox as the
regular messages we send to actors. It is really important to keep the same TraceContext when this happens, since all
possible error messages being logged or any other action taken will be correctly related to the failing request.

### Actor Creation ###

You might be thinking that actor creation happens in the same thread where the call to `ActorRefFactory.actorOf(..)` is
being made, but that is not necessarily true. In fact, a `Create` system message is sent to the newly created actor cell
and it might be execute later depending on whether you are creating a top-level actor or not. Kamon also instrument this
system message to make sure that if an



Propagation through Futures
===========================

### Future's Body ###

In the following piece of code, the body of the future will happen asynchronously on some other thread provided by the
ExecutionContext available in implicit scope, but Kamon will capture the TraceContext available when the future was
created and make it available when executing the future's body.

```scala
val future = TraceRecorder.withNewTraceContext("sample-trace") {
  Future {
    /* Do some sort of calculation */
    "Hello Kamon"
  }
}
```

### Future Callbacks ###

When you transform a future by using map/flatMap/filter and friends or you directly register a
onComplete/onSuccess/onFailure callback on a future, Kamon will capture the TraceContext available transforming the
future and make it available when executing the given callback.

```scala
future
  .map(_.length)
  .flatMap(len ⇒ Future(len.toString))
  .map(s ⇒ TraceRecorder.currentContext)

```

The code snippet above would yield the same TraceContext that was available when creating the future, as well as making
it available during the execution of the maps and flatMap operations.



Finishing a TraceContext
========================

Finishing a TraceContext is pretty easy, just call `TraceRecorder.finish(..)` and the currently available TraceContext
will be closed and stopped from being propagated during the processing of the current event.
