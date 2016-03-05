---
title: Recipe 6. Split and Union a Stream
---
## How to split a stream

You would use a **`split`** when you want to split a stream's tuples among a specified number of streams.

For a more detailed description of `split`, refer to the [Javadoc](http://quarks-edge.github.io/quarks/docs/javadoc/quarks/topology/TStream.html#split-int-quarks.function.ToIntFunction-).

Suppose you have a stream of random integers between 0 (inclusive) and 100 (exclusive), and you would like to separate them by the last digit of each number. For instance, all numbers ending in `8` would be contained in a single stream, and likewise with all numbers ending in `9`. We can easily achieve this using `split`. Let's take a look.

First, let's begin by creating a `DirectProvider` and `Topology`.

{% highlight java %}
  import java.util.List;
  import java.util.Random;
  import java.util.concurrent.TimeUnit;

  import quarks.providers.direct.DirectProvider;
  import quarks.topology.TStream;
  import quarks.topology.Topology;

  public class Test {
    public static void main(String[] args) {
      DirectProvider dp = new DirectProvider();
      Topology top = dp.newTopology("randomNums");
    }
  }
{% endhighlight %}

Now, we can define our low/high random values and random number generator.

{% highlight java %}
  static int RANDOM_LOW = 0;
  static int RANDOM_HIGH = 100;
    
  public static Random getRandom() {
    return new Random();
  }
{% endhighlight %}

The next step is to create a stream of random integers. We use the `poll` method to generate a flow of tuples, where each tuple arrives every 10 milliseconds.

{% highlight java %}
  TStream<Integer> nums = top.poll(() -> 
    getRandom().nextInt(RANDOM_HIGH - RANDOM_LOW) + RANDOM_LOW, 10, TimeUnit.MILLISECONDS);
{% endhighlight %}

We are now ready to split the `nums` stream by the last digit of each tuple. Let's look more closely at the method declaration below.

{% highlight java %}
  java.util.List<TStream<T>> split(int n,
                                   ToIntFunction<T> splitter)
{% endhighlight %}

`split` returns a `List` of `TStream` objects, where each item in the list is one of the resulting output streams. In this case, one stream in the list will contain a flow of integer tuples such that the last digit is a `1`. Another stream will contain a flow of integer tuples such that the last digit is a `2`, and so on.

There are two input parameters. You must specify `n`, the number of output streams, as well as a `splitter` method. `splitter` processes each incoming tuple individually and determines on which of the output streams the tuple will be placed. In this method, you can break down your placement rules into different branches, where each branch returns an integer indicating the index of the output stream in the list.

Going back to our example, let's see how we can use `split` to achieve our goal. We pass in `10` as the first argument, as we want 10 output streams (i.e., a stream for each of: numbers ending in 0, numbers in ending in 1, ... , numbers in 9). Our `splitter` method should then define how tuples will be placed on each of the 10 streams. We use a `switch` statement to check for the result of `num % 10`, which returns the last digit of the the tuple. For example, if we are processing the tuple `11`, the operation `11 % 1` returns `1`. We return that value, meaning that `11` will be placed in the stream at index 1 in the `streams` list. We follow a similar process for the other possible numbers 0 through 9.  

{% highlight java %}
  List<TStream<Integer>> streams = nums.split(10, num -> {
    switch (num % 10) {
      case 0: return 0;
      case 1: return 1;
      case 2: return 2;
      case 3: return 3;
      case 4: return 4;
      case 5: return 5;
      case 6: return 6;
      case 7: return 7;
      case 8: return 8;
      default: return 9;
    }
  });
{% endhighlight %}

Now, we can retrieve the stream containing all numbers ending in `1` by using the standard `List` operation `get`. Likewise, we can retrieve all numbers ending in `7` just as easily.

{% highlight java %}
  TStream<Integer> ones = streams.get(1);
  TStream<Integer> sevens = streams.get(7);
{% endhighlight %}

The `ones` stream when printed would look like:

{% highlight java %}
  11
  81
  51
  81
  21
  31
  1
{% endhighlight %}

As always, we end our application by submitting the `Topology`. Here is the final application:

{% highlight java %}
  import java.util.List;
  import java.util.Random;
  import java.util.concurrent.TimeUnit;

  import quarks.providers.direct.DirectProvider;
  import quarks.topology.TStream;
  import quarks.topology.Topology;

  public class Test {
    static int RANDOM_LOW = 0;
    static int RANDOM_HIGH = 100;
    
    public static Random getRandom() {
      return new Random();
    }
    
    public static void main(String[] args) {
      DirectProvider dp = new DirectProvider();
      Topology top = dp.newTopology("randomNums");
      
      TStream<Integer> nums = top.poll(() -> 
        getRandom().nextInt(RANDOM_HIGH - RANDOM_LOW) + RANDOM_LOW, 
        10, TimeUnit.MILLISECONDS);
      
      List<TStream<Integer>> streams = nums.split(10, num -> {
        switch (num % 10) {
          case 0: return 0;
          case 1: return 1;
          case 2: return 2;
          case 3: return 3;
          case 4: return 4;
          case 5: return 5;
          case 6: return 6;
          case 7: return 7;
          case 8: return 8;
          default: return 9;
        }
      });
      
      TStream<Integer> ones = streams.get(1);     // All numbers ending in 1
      TStream<Integer> sevens = streams.get(7);   // All numbers ending in 7
      ones.print();
      
      dp.submit(top);
    }
  }
{% endhighlight %}


## How to union a stream

You would use a **`union`** when you want to combine multiple streams into a single stream.

For a more detailed description of `union`, refer to the [Javadoc](http://quarks-edge.github.io/quarks/docs/javadoc/quarks/topology/TStream.html#union-quarks.topology.TStream-).

First, let's begin by creating a `DirectProvider` and `Topology`.

{% highlight java %}
  import java.util.HashSet;
  import java.util.Set;

  import quarks.providers.direct.DirectProvider;
  import quarks.topology.TStream;
  import quarks.topology.Topology;

  public class Test {
    public static void main(String[] args) {
      DirectProvider dp = new DirectProvider();
      Topology top = dp.newTopology("unionTest");
    }
  }
{% endhighlight %}

Now, we can create three different streams of strings. The first stream will contain a few lowercase letters, the second will contain a few uppercase letters, and the third will contain a few numbers.

{% highlight java %}
  TStream<String> stream1 = top.strings("a", "b", "c");
  TStream<String> stream2 = top.strings("A", "B", "C");
  TStream<String> stream3 = top.strings("1", "2", "3");
{% endhighlight %}

There are two ways to define a union. You can either union a `TStream` with another `TStream`, or with a set of streams (`Set<TStream<T>>`). In both cases, a single `TStream` is returned containing the tuples that flow on the input stream(s).

Let's look at the first case, unioning a stream with a single stream. We can create a stream of letters by unioning `stream1` with `stream2`.

{% highlight java %}
  TStream<String> letters = stream1.union(stream2);
{% endhighlight %}

When you print the `letters` stream, it will contain all of the strings on each of the input streams, namely:

{% highlight java %}
  a
  b
  c
  A
  B
  C
{% endhighlight %}

If we look at the other case, unioning a stream with a set of streams, we can observe similar results. We'll first create a set containing the three streams we defined earlier. 

{% highlight java %}
  Set<TStream<String>> stream123 = new HashSet<>();
  stream123.add(stream1);
  stream123.add(stream2);
  stream123.add(stream3);
{% endhighlight %}

We can then create a fourth stream with the tuples `"hello"` and `"Quarks"`, and easily create a union of all four streams.

{% highlight java %}
  TStream<String> stream4 = top.strings("hello", "Quarks");

  TStream<String> unionedStream = stream4.union(stream123);   
{% endhighlight %}

`unionedStream` will contain all the tuples from `stream1`, `stream2`, `stream3`, and `stream4`:

{% highlight java %}
  a
  b
  c
  1
  2
  3
  A
  B
  C
  hello
  Quarks 
{% endhighlight %}

As always, we end our application by submitting the `Topology`. Here is the final application:

{% highlight java %}
  import java.util.HashSet;
  import java.util.Set;

  import quarks.providers.direct.DirectProvider;
  import quarks.topology.TStream;
  import quarks.topology.Topology;

  public class Test {
    public static void main(String[] args) {
      DirectProvider dp = new DirectProvider();
      Topology top = dp.newTopology("unionTest");
      
      TStream<String> stream1 = top.strings("a", "b", "c");
      TStream<String> stream2 = top.strings("A", "B", "C");
      TStream<String> stream3 = top.strings("1", "2", "3");

      TStream<String> letters = stream1.union(stream2);   // Union one stream with another stream
      letters.print();
      
      Set<TStream<String>> stream123 = new HashSet<>();
      stream123.add(stream1);
      stream123.add(stream2);
      stream123.add(stream3);
      
      TStream<String> stream4 = top.strings("hello", "Quarks");

      TStream<String> unionedStream = stream4.union(stream123);   // Union one stream with a set of streams
      unionedStream.print();
      
      dp.submit(top);
    }
}
{% endhighlight %}
