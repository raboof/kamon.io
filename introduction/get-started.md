---
title: Kamon | Get Started
layout: documentation
---

Get Started with Kamon
======================

Kamon is distributed as a core and a set of modules that you include in your application classpath. This modules contain
all the required pointcuts and advices (yeap, Kamon uses Aspectj!) for instrumenting Akka actors message passing,
dispatchers, futures, Spray and Play components and much more.

To get started just follow this steps:


First: Include the modules you want in your project.
----------------------------------------------------

All Kamon components are available through Sonatype and Maven Central and no special repositories need to be configured.
If you are using SBT, you will need to add something like this to your build definition:

```scala
libraryDependencies += "io.kamon" %% "kamon-core" % "0.3.5"
```

Then, add any additional module you need:

* kamon-core
* kamon-spray
* kamon-play
* kamon-statsd
* kamon-newrelic
* kamon-datadog
* kamon-log-reporter
* kamon-system-metrics
* kamon-akka-remote <span class="label label-warning">experimental</span></li>

### Compatibility Notes: ###
* 0.3.x releases are compatible with Akka 2.3, Spray 1.3, Play 2.3 and Scala 2.11.x/2.10.x
* 0.2.x releases are compatible with Akka 2.2, Spray 1.2, Play 2.2 and Scala 2.10.x


Second: Start your app with the AspectJ Weaver
----------------------------------------------

Starting your app with the AspectJ weaver is dead simple, just add the `-javaagent` JVM startup parameter pointing to
the weaver's file location and you are done:

```
-javaagent:/path-to-aspectj-weaver.jar
```

In case you want to keep the AspectJ related settings in your build and enjoy using `run` from the console, take a look
at the [sbt-aspectj] plugin - in particular the [load-time weaving example].


### Optional: Register the Metrics Extension ###

If you correctly configured your application to start using the AspectJ Weaver agent then the Metrics extension will be
loaded when the first instrumentation point that stores metrics gets executed. If you don't configure the agent properly
then the Metrics extension will log a warning when it is loaded, but if the agent is not present and you are not
recording any metrics manually then the Metrics wont load and you wont see the warning. By adding the Metrics extension
to your application configuration as shown bellow, you ensure that it is loaded when your actor system is started and
the warning will be displayed is necessary:

```
akka {
  extensions = ["kamon.metric.Metrics"]
}
```

Third: Enjoy!
-------------

Refer to module's documentation to find out more about the core concepts of [tracing] and [metrics], and learn how to
report your metrics data to external services like [StatsD], [Datadog] and [New Relic].


[sbt-aspectj]: https://github.com/sbt/sbt-aspectj/
[load-time weaving example]: https://github.com/sbt/sbt-aspectj/tree/master/src/sbt-test/weave/load-time/
[tracing]: /core/tracing/core-concepts/
[metrics]: /core/metrics/core-concepts/
[logging]: /core/tracing/logging/
[StatsD]: /backends/statsd/
[Datadog]: /backends/datadog/
[New Relic]: /backends/newrelic/
