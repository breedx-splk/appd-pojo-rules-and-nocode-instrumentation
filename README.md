
# Hello POJO, meet nocode

Sometimes, you just have to do it the hard way.

- [What's the Challenge?](#whats-the-challenge)
- [Declarative Instrumentation](#declarative-instrumentation)
- [The OTel Java Way](#the-otel-java-way)
- [The Splunk OTel Way](#the-splunk-otel-way)
- [Migrating from AppDynamics POJO rules](#migrating-from-appdynamics-pojo-rules)
  - [What's in a POJO Rule?](#whats-in-a-pojo-rule)
    - [Other Class Matches](#other-class-matches)
      - [Matching by Interface](#matching-by-interface)
      - [Matching by Superclass](#matching-by-superclass)
      - [Matching by Annotation](#matching-by-annotation)
    - [Other Matching Operations](#other-matching-operations)
      - ["Starts With"](#starts-with)
      - ["Ends With"](#ends-with)
      - ["Contains"](#contains)
      - ["Matches Reg Ex"](#matches-reg-ex)
      - ["Is in List"](#is-in-list)
      - ["Is Not Empty"](#is-not-empty)
- [Conclusion](#conclusion)

# What's the Challenge?

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
indeed be uncommon, it happens. This typically falls into one of three common scenarios:

- Third-party precompiled software without modifiable source.
- Organizational doctrine that prohibits observability teams from modifying source code.
- Legacy code that is too risky or even impossible to rebuild or redeploy.

In these extreme cases, when auto-instrumentation doesn't suffice and manual
instrumentation isn't possible, what can we do?

# Declarative Instrumentation

The early generation of software observability vendors all offered their own
unique way of allowing users to define instrumentation points declaratively.
While slightly different in approach, they did more or less the same thing --
they allowed instrumentation agents to create telemetry based on user-defined
runtime configuration.

For example, AppDynamics offers a capability to instrument "plain ol' Java objects",
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

# The OTel Java Way

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

# The Splunk OTel Way

To provide additional user capabilities, the Splunk distribution of
OpenTelemetry Java Instrumentation offers ["nocode instrumentation"](https://github.com/signalfx/splunk-otel-java/tree/main/instrumentation/nocode).
It provides a highly flexible ability to declare instrumentation
points via class and method, but also allows for:

- customizing the span name
- customizing the span status field
- adding custom attributes to the span

All of the above are configured with [JEXL](https://commons.apache.org/proper/commons-jexl/reference/syntax.html) syntax.

This feature is marked "under development" and is subject to breaking changes or
removal. There is [some
effort](https://github.com/open-telemetry/opentelemetry-java-instrumentation/pull/13590)
to contribute this instrumentation to OpenTelemetry Java Instrumentation upstream,
but this is very much still a work in progress.

Because JEXL performs evaluation of arbitrary user expressions, it is both
powerful and dangerous. A malicious use of JEXL via nocode instrumentation
could, in the worst case, lead to JVM shutdown or leaking of sensitive data.

# Migrating from AppDynamics POJO rules

Users who are migrating from AppDynamics to Splunk Observability Cloud may wish
to migrate their existing legacy POJO definitions. This can help to provide
coverage in areas that are not readily covered with existing instrumentation in
the Splunk OTel Java agent.

It's important to first acknowledge that these two observability platforms use
very different data models. AppDynamics has a bespoke data model centered around
"Business Transactions" (BTs), while OpenTelemetry-based systems are rooted in
distributed traces, or simply traces, which are comprised of spans.

> BTs and traces have a lot in common, but they are not the same thing!

We emphasize this, because POJO rules are intended to trigger BT creation, while
`nocode` instrumentation intends to create spans.

## What's in a POJO Rule?

This dialog allows you to define a POJO rule in the AppDynamics controller:

<img width="596" height="674" alt="image" src="https://github.com/user-attachments/assets/21f55d22-6fc5-4f09-b5a1-491e2f5ac392" />

It allows the user to choose what classes and methods should trigger the
rule, and how those should be matched.

If you simply want to match exactly on a given class and method name, the
mapping to `nocode` is straightforward. For example, a class named `com.example.MyHotClass` with
a method of `doSomething()` would appear like this in AppDynamics:

<img width="375" height="223" alt="image" src="https://github.com/user-attachments/assets/a1cb8344-2a36-40bc-8fb1-7ce02c9b7c72" />

and its `nocode` YAML definition should be:
```yaml
- class: com.example.MyHotClass
  method: doSomething
  span_kind: CLIENT
```

_Note: It is highly recommended to provide the `span_kind`. This tells the Splunk Observability Cloud backend what [kind of span](https://opentelemetry.io/docs/concepts/signals/traces/#span-kind) this is._

### Other Class Matches

Sometimes, POJOs are matched not purely on their exact class name,
but on other class characteristics:

<img width="283" height="223" alt="image" src="https://github.com/user-attachments/assets/3cae5791-3323-4e8f-b802-2bede254625e" />

The [`nocode` documentation](https://github.com/signalfx/splunk-otel-java/tree/main/instrumentation/nocode#more-complex-classmethod-selection) covers these additional, more complicated cases, but we will provide examples here too.

Please note that Java class names should always be fully qualified with the
complete package name.

> Note: Regular expressions in YAML should usually be surrounded with single quotes. Periods in package names should be escaped with a backslash.

#### Matching by Interface

<img width="385" height="117" alt="image" src="https://github.com/user-attachments/assets/67ae5fa6-fee2-4db0-8e52-f1e5b390692e" />

You may have POJO rules that apply to all classes that implement
an interface. When this is the case, use the `super_type` selector
in your `nocode` definition. For example, to instrument all
classes that implement the `com.example.SelectionStrategy` interface,
use the following:

```yaml
- class:
    super_type: com.example.SelectionStrategy
  method: choose
  span_kind: CLIENT
```

> _Note: It is not currently possible to perform non-exact matches with `super_type` class selectors in `nocode`_

#### Matching by Superclass

Similar to the interface match (above), some POJO rules might match
on a classes parentage or ancestry via "Super Class".

<img width="378" height="116" alt="image" src="https://github.com/user-attachments/assets/2235323c-d788-42cf-b8d3-a54a229fb579" />

In `nocode`, this is also accomplished with the `super_type` matcher.
For example, to match all subclasses of `com.example.SuperServlet`, you 
can use the following `nocode` YAML snippet:

```yaml
- class:
    super_type: com.example.SuperServlet
  method: serve
  span_kind: SERVER
```

> _Note: It is not currently possible to perform non-exact matches with `super_type` class selectors in `nocode`_

#### Matching by Annotation

<img width="377" height="111" alt="image" src="https://github.com/user-attachments/assets/711f3d68-5f6a-40c8-b907-ed7ae056991a" />

AppDynamics POJO rule definitions allow users to select classes that have a certain annotation. For example, you might have targeted a class that looks like this:

```java

@MakeBizTransaction
class MyUsefulClass { 
  ...
}
```

where the `@MakeBizTransaction` is fully qualified to
`com.example.MakeBizTransaction`. At the time of this writing, there is no means
of matching classes by annotation in `nocode` YAML expressions. This remains an
item for future enhancement.

> _Note: Annotations must have a [RUNTIME retention
> policy](https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/RetentionPolicy.html#RUNTIME)
> to be useful here._

### Other Matching Operations

In addition to the precise "Equals" match, AppDynamics POJO definitions
can use several other, more flexible matching predicates. This section
will describe those and how to map them to `nocode` yaml definitions:

<img width="290" height="353" alt="image" src="https://github.com/user-attachments/assets/1968438e-0d3a-46db-adbe-006516f651e0" />

#### "Starts With"

<img width="383" height="53" alt="image" src="https://github.com/user-attachments/assets/cfdbfdba-9f24-495e-9cf8-c9d0fcf1f3aa" />

To perform a "Starts With" match in nocode, use a regular expression with the
caret ('^') to match the start of the string, and use a wildcard through the end
of the string. For example, to match any class that starts with the name
`com.example.Foo`, the `nocode` equivalent should be:

```yaml
- class:
    name_regex: '^com\.example\.Foo.*'
  method: bar
  span_kind: SERVER
```

This definition matches both `com.example.FooBarImpl` and
`com.example.FoodFight` classes.

#### "Ends With"

<img width="386" height="51" alt="image" src="https://github.com/user-attachments/assets/66d27913-706b-4f24-a679-3bce599361a1" />

To perform an "Ends With" match in nocode, use a regular expression that starts
with a wildcard, then contains the desired string, and ends with the `$` end of
string marker. For example, to match any class that ends with the name
`common.Util`, the `nocode` equivalent should be:

```yaml
- class:
    name_regex: '.*common\.Util$'
  method: beep
  span_kind: SERVER
```

This definition matches both `com.example.common.Util` and `com.example.uncommon.Util`, but it would not match `com.example.common.Utilities`.

#### "Contains"

<img width="388" height="50" alt="image" src="https://github.com/user-attachments/assets/890ab424-f2e8-493b-8682-995cf28860c2" />

A "Contains" expression essentially combines the above "starts with" and "ends
with" matches. Simply include a wildcard `.*` prefix and suffix around the
desired string. For example, to match any class that contains the string
`FilterFactory`, you could use the following `nocode` YAML expression:

```yaml
- class:
    name_regex: '.*FilterFactory.*'
  method: filter
  span_kind: SERVER
```

This would match `com.example.FilterFactory` and `com.example.AbstractFilterFactoryBaseImpl`.

#### "Matches Reg Ex"

The AppD "Matches Reg Ex" is the same as the `nocode` `name_regex`.

Because over-matching with overly broad match expressions could generate
undesired results, regular expressions should always be used with care.

#### "Is in List"

<img width="383" height="75" alt="image" src="https://github.com/user-attachments/assets/971155bd-c916-48ef-99e3-053dcca3ad6d" />

The "Is in List" matcher allows several class names to be matched.
For example, you might want to match both `com.example.Foo` and `com.example.Bar`
POJOs. To do this with `nocode`, you can leverage the `or` logic operator :

```yaml
- class:
    or:
      - name: com.example.Foo
      - name: com.example.Bar
  method: bedazzle
  span_kind: SERVER
```

#### "Is Not Empty"

<img width="172" height="49" alt="image" src="https://github.com/user-attachments/assets/f5b81409-a7b4-4759-a9b4-4cb5a25f37be" />

I have no idea what this is, and I think it's unlikely that you have POJO rules that leverage this. If you do,
please [reach out and let me know](jplumb@cisco.com).

### Inverting the Method Name

The AppDynamics POJO rule creation dialog contains a gear icon that hides the NOT capability:

<img width="449" height="78" alt="image" src="https://github.com/user-attachments/assets/48a2e697-b298-4875-82df-fc571d710b37" />

Choosing this allows you to invert the match for the method name. In other words, match all methods on the matched class(es) that do NOT satisfy the selection criteria. In `nocode`, you can use the `not` operator to do the same thing. For example, to match all methods whose name is not `swizzle`:

```yaml
- class: com.example.MyClass
  method: 
    not:
      name: swizzle
  span_kind: SERVER
```

## What About Splitting?

The "Add Rule" screen also has a section that allows AppDynamics users to
split Business Transactions. 

<img width="440" height="122" alt="image" src="https://github.com/user-attachments/assets/e46b0033-6b56-4e6b-bb80-5558dff2997a" />

Because the OpenTelemetry and Splunk Observability Cloud data models use transactions and spans, Business Transactions are not applicable. Simply put, there is no way to split a "Business Transaction" in `nocode` because there are no Business Transactions. All Spans will either be the root span of a new Trace, or will be created within the existing trace context.

### `nocode` for existing spans

If you wish to add attributes to the current span instead of creating a new
span, `nocode` has a `current_span` directive. See [the
documentation](https://github.com/signalfx/splunk-otel-java/tree/main/instrumentation/nocode#adding-attributes-to-the-current-span)
for additional details about this feature.

# Conclusion

Sometimes, you just have to do it the hard way. Fortunately, there are
options in the form of declarative instrumentation to help in these
situations.

The Splunk Distribution of OpenTelemetry Java Instrumentation provides a
powerful `nocode` instrumentation module that can be configured to cover
most of the common use cases from the AppDynamics POJO rule definitions.
This allows OpenTelemetry spans to be created for user code or third-party
libraries for which there is no existing instrumentation.

The mappings described in this article can be leveraged by users who
are migrating from AppDynamics to Splunk Observability Cloud and have
existing gaps that need instrumentation coverage for continuity. A word of caution: Declarative instrumentation, like `nocode`, should usually be used as a last resort after determining that other instrumentation doesn't exist or that manual instrumentation cannot be added.

Caution should also be used when defining these rules, because an improperly
written rule could have negative impact on an application. At worst, the
evaluation of an arbitrary Java expression could cause the application to
terminate unexpectedly or to degrade performance. Another mistake might generate
unwanted volumes of telemetry.
