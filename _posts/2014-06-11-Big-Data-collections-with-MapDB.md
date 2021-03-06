---
layout: post
category : pages
tags : [Java, MapDB]
disqusid : mapdb
---

This article gives a short overview over the open source software&nbsp;<a href="http://www.mapdb.org/">MapDB</a> 
which is now in version 1.0.3. 
    
What is MapDB?
==============
Original designed as a storage engine for an astronomical desktop application it had two design goals <b>minimal overhead</b> and <b>simplicity</b>. Over the time the engine had evolved and the third goal <b>provide an alternative Java memory model</b>&nbsp;was added. So now it is a storage engine which is specialized for big data collections and for that has some cool features.<br />
For example:
<ul>
        <li>Write to Heap, OffHeap, File or TempFile</li>
        <li>Synchronization of Maps/TreeMaps/Sets and Queues&nbsp;</li>
        <ul>
            <li>Maps can also be build with composite keys</li>
            <li>bidirectional maps</li>
            <li>synchronization between maps (in case you have a 1-N association)</li>
        </ul>
        <li>Caching</li>
        <ul>
            <li>expiration on disk usage, access or write time</li>
        </ul>
        <li>Compression</li>
        <li>Faceting aka Histogram</li>
        <li>Simulated Auto-Increment</li>
        <li>Transactions (Note: a single transaction can only be used once)</li>
        <li>Querying</li>
    </ul>
    
<h2>Small Example</h2>
The following example shows the simplicity in the context of IoT where i put 10 million temperature values into a collection which is backed by an off-heap and group the values into five groups (cold, fresh, warm, hot and burns). For filling the cache I also use auto increment.

{% highlight java linenos %}
    public class TemperatureRepository {
    private final Atomic.Long keyinc;
    private ConcurrentHashMap<String, Long> histogram;
    private HTreeMap<Long, Integer> temperatureMap;

    public TemperatureRepository() {
        //Create off-heap memory cache
        temperatureMap = DBMaker.newCache(1.0);

        //Get Autoincrement counter
        DB db = new DB(temperatureMap.getEngine());
        keyinc = db.getAtomicLong("map_temp");

        // histogram, category is a key, count is a value
        histogram = new ConcurrentHashMap<String, Long>(); //any map will do

        // bind histogram to primary map
        // we need function which returns category for each map entry
        Bind.histogram(temperatureMap, histogram, (key, value) -> {
            String ret = null;

            if (value < 0) {
                ret = "cold";
            } else if (value < 10) {
                ret = "fresh";
            } else if (value < 20) {
                ret = "warm";
            } else if (value < 30) {
                ret = "hot";
            } else {
                ret = "burns";
            }
            return ret;
        });
    }

    public void add(int temperature) {
        temperatureMap.put(keyinc.incrementAndGet(), temperature);
    }

    public void printHistogram() {
        System.out.println(histogram);
    }

    public static void main(String[] args) {
        TemperatureRepository temperatureRepository = new TemperatureRepository();
        new Random().ints(-10,40).parallel().limit(1_000_000).forEach(e-> temperatureRepository.add(e));
        temperatureRepository.printHistogram();
    }
}
{% endhighlight %}

Fazit
=====
Until now I haven't had the chance to use MapDB in a productive environment but on our playground 
at&nbsp;<a href="http://www.rapidpm.org/">www.rapidpm.org</a>&nbsp;it makes a very good impression.

