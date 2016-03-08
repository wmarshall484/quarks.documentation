---
title: Recipe 1. Hello Quarks!
---

In true "hello world" fashion, the most simple Quarks application is one which sends the single Java String "Hello Quarks!" from a single source, and prints it to standard output. Quarks makes this easy by providing a suite of utility methods for creating data sources, one of which is the *Topology.strings* method. 

## The Strings Method
{{site.data.alerts.note}} A data source is a stream which has no input streams, and generates its own data. {{site.data.alerts.end}}

*Topology.strings* creates a stream of Java Strings by allowing the user to provide one or more Strings to be used as data items.

To illustrate this, let's start with the following template:

``` java
    public static void main(String[] args) {
        DirectProvider dp = new DirectProvider();
        Topology top = dp.newTopology();
    }
```

As mentioned in the Getting Started Guide, the *DirectProvider* allows the user to submit the final Quarks application, and the *Topology* enables the user to define data sources. Invoking *Topology.strings* defines a stream:

``` java
    public static void main(String[] args) {
        DirectProvider dp = new DirectProvider();
        Topology top = dp.newTopology();
        TStream<String> helloStream = top.strings("Hello Quarks!");
    }
```

## Printing to Output

To print each data item of a TStream to the screen, the *TStream.print* method creates a sink prints the string representation of each data item to standard output:

{{site.data.alerts.note}} A data sink is an operation on a stream which produces no output streams. Operations such as <i>TStream.print</i> are data sinks. {{site.data.alerts.end}}

``` java
    public static void main(String[] args) {
        DirectProvider dp = new DirectProvider();
        Topology top = dp.newTopology();
        TStream<String> helloStream = top.strings("Hello Quarks!");
		helloStream.print();
    }
```

## Submitting the Application
The only remaining step is to submit the application, which is performed my the *DirectProvider*:

``` java
    public static void main(String[] args) {
        DirectProvider dp = new DirectProvider();
        Topology top = dp.newTopology();
        TStream<String> helloStream = top.strings("Hello Quarks!");
        helloStream.print();
        dp.submit(top);
    }
```

After running the application, the output should simply be the expected "Hello Quarks!":

```
Hello Quarks!
```




