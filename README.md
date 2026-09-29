
# Hello POJO, meet nocode

Sometimes, you just have to do it the hard way.

Instrumentation agents are highly advanced bits of software that manipulate
bytecode, intercept library calls, and wrap and observe other software in order
to create telemetry. Their instrumentation modules are created specifically for
common, popular frameworks and libraries, and the coverage for most use cases is
astounding! Furthermore, we're seeing more and more libraries adopt
OpenTelemetry as a native, first-class feature. However, there are occasionally
still some times when off-the-shelf instrumentation isn't complete enough, or it
doesn't provide exactly what a user wants.

With modern standards like OpenTelemetry, the first bit of guidance would be to
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
instruemtnation modules. In some organizations, the tribal knowledge of how to
even build

AppDynamics offers a capability to instrumentation "plain ol' Java objects",
often simply referred to as "POJO", with
[custom match rules](https://help.splunk.com/en/appdynamics-saas/application-performance-monitoring/26.4.0/configure-instrumentation/transaction-detection-rules/custom-match-rules/java-business-transaction-detection/pojo-entry-points/about-pojo-custom-match-rules).
This

AppDynamics POJO instrumentation and Splunk OTel Java

* the dangers of using this stuff
* 