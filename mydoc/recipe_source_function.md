---
title: Recipe 2. Writing a Source Function
---
In the previous [Hello Quarks!](recipe_hello_quarks) example, we create a data source which only generates a single Java String and prints it to output. Yet Quarks sources support the ability generate any data type as a source, not just primitive Java types such as Strings. Moreover, because the user supplies the code which generates the data, the user has complete flexibility for *how* the data is generated. This recipe demonstrates how a user could write such a custom data source.

## Custom Source: Reading the Lines of a Web Page
{{site.data.alerts.note}} Quarks' API provides convenience methods for performing HTTP requests. For the sake of example we are writing a HTTP data source manually, but in principle there are easier methods. {{site.data.alerts.end}}

One example of a custom data source could be retrieving the contents of a web page and printing each line to output. For example, the user could be querying the Yahoo Finance website for the most recent stock price data of Bank of America, Cabot Oil & Gas, and Freeport-McMoRan Inc:

``` java
    public static void main(String[] args) throws MalformedURLException {
        DirectProvider dp = new DirectProvider();
        Topology top = dp.newTopology();
        
        final URL url = new URL("http://finance.yahoo.com/d/quotes.csv?s=BAC+COG+FCX&f=snabl");
	}
```

Given the correctly formatted URL to request the data, we can use the *Topology.source* method to generate each line of the page as a data item on the stream. 

``` java
    public static void main(String[] args) throws MalformedURLException {
        DirectProvider dp = new DirectProvider();
        Topology top = dp.newTopology();
        
        final URL url = new URL("http://finance.yahoo.com/d/quotes.csv?s=BAC+COG+FCX&f=snabl");
        
        TStream<String> linesOfWebsite = top.source(() -> {
            List<String> lines = new ArrayList<>();
            
            try {
                InputStream is = url.openStream();
                BufferedReader br = new BufferedReader(
                        new InputStreamReader(is));
                
                for(String s = br.readLine(); s != null; s = br.readLine())
                    lines.add(s);

            } catch (Exception e) { e.printStackTrace(); }
            
            return lines;
        });
	}
```

*Topology.source* takes a Java Supplier that returns an Iterable. In this case, the iterable is an ArrayList which contains each line of the retrieved website. If we print the *linesOfWebsite* stream to standard output and run the application, we can see that it correctly generates the data and feeds it into the Quarks runtime:

``` java
    public static void main(String[] args) throws MalformedURLException {
        DirectProvider dp = new DirectProvider();
        Topology top = dp.newTopology();
        
        final URL url = new URL("http://finance.yahoo.com/d/quotes.csv?s=BAC+COG+FCX&f=snabl");
        
        TStream<String> source = top.source(() -> {
            List<String> lines = new LinkedList<>();
            
            try {
                InputStream is = url.openStream();
                BufferedReader br = new BufferedReader(
                        new InputStreamReader(is));
                
                for(String s = br.readLine(); s != null; s = br.readLine())
                    lines.add(s);

            } catch (Exception e) { e.printStackTrace(); }
            
            return lines;
        });
        
        source.print();
        dp.submit(top);
    }
```

Output:

```
"BAC","Bank of America Corporation Com",13.150,13.140,"12:00pm - <b>13.145</b>"
"COG","Cabot Oil & Gas Corporation Com",21.6800,21.6700,"12:00pm - <b>21.6775</b>"
"FCX","Freeport-McMoRan, Inc. Common S",8.8200,8.8100,"12:00pm - <b>8.8035</b>"
```