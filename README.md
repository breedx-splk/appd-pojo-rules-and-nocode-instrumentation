
# Hello POJO, meet nocode

Sometimes, you just have to do it the hard way.

## What's the Challenge?

Instrumentation agents are highly advanced bits of software that manipulate
bytecode, intercept library calls, and wrap and observe other software in order
to create telemetry. Their instrumentation modules are created specifically for
common, popular frameworks and libraries, and the coverage for most use cases is
astounding! Furthermore, we're seeing more and more libraries adopt
OpenTelemetry as a native, first-class feature. However, there are occasionally
still some times when off-the-shelf instrumentation isn't complete enough, or it
doesn't provide exactly what a user wants.

With modern standards like OpenTelemetry, the first bit of guidance is to
leverage [manual
instrumentation](https://opentelemetry.io/docs/languages/java/instrumentation/#manual-instrumentation).
In OTel, manual instrumentation allows software developers to insert annotations
or wire up API calls in order to create the app-specific telemetry that they
need. Because this manual instrumentation is created with code, it does increase
the developer maintenance burden and, if not done with care, can clutter up the
readability of core business logic.

Sometimes, manual instrumentation isn't possible or practical. While this should
indeed be uncommon, it happens. Users sometimes need to operate 3rd-party,
closed-source, or expensive or niche software for which there exist no good
instrumentation modules. In some organizations, the tribal knowledge of how to
even build internal software has been lost to time and isn't possible. In other
companies, the observability team may not be permitted to make code-level
changes or even talk with the development team!

In these extreme cases, when auto-instrumentation doesn't suffice and manual
instrumentation isn't possible, what can we do?

## Declarative Instrumentation

The early generation of software observability vendors all oferred their own
unique way of allowing users to define instrumentation points declaratively.
While slightly different in approach, they did more or less the same thing --
they allowed instrumentation agents to create telemetry based on user-defined
runtime configuration.

For example, AppDynamics offers a capability to instrumentation "plain ol' Java objects",
often simply referred to as "POJO", with
[custom match rules](https://help.splunk.com/en/appdynamics-saas/application-performance-monitoring/26.4.0/configure-instrumentation/transaction-detection-rules/custom-match-rules/java-business-transaction-detection/pojo-entry-points/about-pojo-custom-match-rules). These rules are defined in the
AppD Controller UI, which then passes this configuration down to the agents
via its bespoke, proprietary protocol. The agent will then create
AppD "Business Transactions" when these rules are matched.

Similarly, the bespoke New Relic agent has ["XML
Instrumentation"](https://docs.newrelic.com/docs/apm/agents/java-agent/custom-instrumentation/java-instrumentation-xml/).
With this, users could write lots of angle brackets into a file in order to
define how "Transactions" might be created. This file is then passed to the
agent via configuration.

Naturally, DataDog created their own similar offering, in the form of its
[`dd.trace.methods`
configuration](https://github.com/DataDog/dd-trace-java/pull/311). Just like the
other implementations, this allows users to define a set of classes and methods
that should be instrumented, but in this case for tracing.

## The OTel Java Way

The OpenTelemetry Java Agent also provides a flexible way to capture
telemetry at the method level, via its ["methods
instrumentation"](https://github.com/open-telemetry/opentelemetry-java-instrumentation/tree/main/instrumentation/methods).
This is a first-class instrumentation that is automatically included
in the OpenTelemetry distribution. Configuration of the methods instrumentation can be provided through environment variables and system properties,
but because it can be quite verbose, using declarative config (yaml)
is recommended.

While simple and useful, the instrumentation is quite limited. For instance, it
doesn't provide a way to discriminate overloaded methods, nor does it allow
customizing the name of the new span. There is also no ability to surface method
return value or method parameter values.

Fortunately, the OTel Java Agent is open source, and pull requests that
enhance the instrumentation capabilities are welcome.

## The Splunk OTel Way

To provide additional user capabilities, the Splunk distribution of
OpenTelemetry Java Instrumentation offers ["nocode instrumentation"](https://github.com/signalfx/splunk-otel-java/tree/main/instrumentation/nocode).
It provides a highly flexible ability to declare instrumentation
points via class and method, but also allows for:

* customizing the span name
* customizing the span status field
* adding custom attributes to the span

All of the above are configured with [JEXL](https://commons.apache.org/proper/commons-jexl/reference/syntax.html) syntax.

This feature is marked "under development" and is subject to breaking changes or
removal. There is [some
effort](https://github.com/open-telemetry/opentelemetry-java-instrumentation/pull/13590)
to contribute this instrumentation to OpenTelemery Java Instrumentation upstream,
but this is very much still a work in progress.

tbd
experimental - "under development"

AppDynamics POJO instrumentation and Splunk OTel Java

## Migrating from AppDynamics POJO rules

## Conclusion

* the dangers of using this stuff
* 