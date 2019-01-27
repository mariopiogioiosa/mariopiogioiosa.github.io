---
id: 136
title: 'Java 8 Comparator &#8211; How to sort a List'
date: 2016-11-21T23:10:17+00:00
author: Mario Pio Gioiosa
layout: post
guid: http://reversecoding.net/?p=136
permalink: /java-8-comparator-how-to-sort-a-list/
hefo_before:
  - "0"
hefo_after:
  - "0"
categories:
  - Java 8
tags:
  - comparator
  - Java
  - Java 8
  - sort a list
  - sorting
---
<p style="text-align: justify;">
  In this article, we're going to see several examples on how to sort a <em>List</em> in Java 8.
</p>

## 1. Sort a List of String alphabetically

<pre class="lang:default decode:true" title="Sort List of String alphabetically">List&lt;String&gt; cities = Arrays.asList(
       "Milan",
       "london",
       "San Francisco",
       "Tokyo",
       "New Delhi"
);
System.out.println(cities);
//[Milan, london, San Francisco, Tokyo, New Delhi]

cities.sort(String.CASE_INSENSITIVE_ORDER);
System.out.println(cities);
//[london, Milan, New Delhi, San Francisco, Tokyo]

cities.sort(Comparator.naturalOrder());
System.out.println(cities);
//[Milan, New Delhi, San Francisco, Tokyo, london]</pre>

<p style="text-align: justify;">
  By purpose, we've written London with &#8220;L&#8221; in low-case to better highlight difference between <em><a href="https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html#naturalOrder--" target="_blank" rel="noopener">Comparator.naturalOrder()</a> </em>that<em> </em>returns a <a href="https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html" target="_blank" rel="noopener"><em>Comparator</em></a> that sorts by placing capital letters first and <em><a href="http://docs.oracle.com/javase/8/docs/api/java/lang/String.html#CASE_INSENSITIVE_ORDER">String.CASE_INSENSITIVE_ORDER</a> </em>that returns a case-insensitive <em>Comparator</em>.
</p>

<p style="text-align: justify;">
  Basically, in Java 7 we were using <em>Collections.sort()</em> that was accepting a <em>List</em> and, eventually, a <em>Comparator</em> &#8211;  in Java 8 we have the new <em><a href="http://docs.oracle.com/javase/8/docs/api/java/util/List.html#sort-java.util.Comparator-" target="_blank" rel="noopener">List.sort()</a></em> that accepts a <em>Comparator</em><em>.</em>
</p>

## 2. Sort a List of Integer

<pre class="lang:default decode:true">List&lt;Integer&gt; numbers = Arrays.asList(6, 2, 1, 4, 9);
System.out.println(numbers); //[6, 2, 1, 4, 9]

numbers.sort(Comparator.naturalOrder());
System.out.println(numbers); //[1, 2, 4, 6, 9]</pre>

## 3. Sort a List by String field

<p style="text-align: justify;">
  Let's suppose we've our <em>Movie</em> class and we want to sort our <em>List</em> &#8220;by title&#8221;. We can use <a href="https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html#comparing-java.util.function.Function-" target="_blank" rel="noopener"><em>Comparator.comparing()</em></a> and pass a function that extracts the field to use for sorting &#8211; title &#8211; in this example.
</p>

<pre class="lang:default decode:true">List&lt;Movie&gt; movies = Arrays.asList(
        new Movie("Lord of the rings"),
        new Movie("Back to the future"),
        new Movie("Carlito's way"),
        new Movie("Pulp fiction"));

movies.sort(Comparator.comparing(Movie::getTitle));

movies.forEach(System.out::println);</pre>

The output will be:

<pre class="lang:diff decode:true">Movie{title='Back to the future'}
Movie{title='Carlito's way'}
Movie{title='Lord of the rings'}
Movie{title='Pulp fiction'}</pre>

<p style="text-align: justify;">
  As you've probably noticed we haven't passed any <em>Comparator </em>but the <em>List</em> is correctly sorted. That's because of the title &#8211; the extracted field &#8211; that is a <em>String</em> and <em>String</em> implements <em>Comparable</em> interface. If you peek at <em>Comparator.comparing()</em> implementation you will see that it calls <strong><em>compareTo</em></strong> on the extracted key.
</p>

<pre class="lang:default decode:true">return (Comparator&lt;T&gt; & Serializable)
            (c1, c2) -&gt; keyExtractor.apply(c1).compareTo(keyExtractor.apply(c2));</pre>

## 4. Sort a List by double field

<p style="text-align: justify;">
  In a similar way, we can use <a href="https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html#comparingDouble-java.util.function.ToDoubleFunction-" target="_blank" rel="noopener"><em>Comparator.comparingDouble()</em></a> for comparing <em>double</em> value. In the example, we want to order our <em>List</em> of <em>Movie</em> by rating, from the highest to the lowest.
</p>

<pre class="lang:default decode:true">List&lt;Movie&gt; movies = Arrays.asList(
        new Movie("Lord of the rings", 8.8),
        new Movie("Back to the future", 8.5),
        new Movie("Carlito's way", 7.9),
        new Movie("Pulp fiction", 8.9));

movies.sort(Comparator.comparingDouble(Movie::getRating)
                      .reversed());

movies.forEach(System.out::println);</pre>

<p style="text-align: justify;">
  We used <a href="https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html#reversed--" target="_blank" rel="noopener"><em>reversed</em></a> function on the <em>Comparator</em> in order to invert default natural-order that is from lowest to highest. <em>Comparator.comparingDouble()</em> uses <em>Double.compare()</em> under the hood.
</p>

<p style="text-align: justify;">
  If you need to compare <em>int</em> or <em>long</em> you can use <em>comparingInt()</em> and <em>comparingLong() </em>respectively<em>.</em>
</p>

## 5. Sort a List with custom Comparator

In the previous examples we haven't specified any Comparator since it wasn't necessary but let's see an example in which we define our own _Comparator_. Our _Movie_ class has a new field &#8211; &#8220;starred&#8221; &#8211; set using the third constructor parameter. In the example, we want to sort the list so that we have starred movie at the top of the _List. _

<pre class="lang:default decode:true">List&lt;Movie&gt; movies = Arrays.asList(
        new Movie("Lord of the rings", 8.8, true),
        new Movie("Back to the future", 8.5, false),
        new Movie("Carlito's way", 7.9, true),
        new Movie("Pulp fiction", 8.9, false));

movies.sort(new Comparator&lt;Movie&gt;() {
    @Override
    public int compare(Movie m1, Movie m2) {
        if(m1.getStarred() == m2.getStarred()){
            return 0;
        }
        return m1.getStarred() ? -1 : 1;
     }
});

movies.forEach(System.out::println);</pre>

The result will be:

<pre class="lang:diff decode:true">Movie{starred=true, title='Lord of the rings', rating=8.8}
Movie{starred=true, title='Carlito's way', rating=7.9}
Movie{starred=false, title='Back to the future', rating=8.5}
Movie{starred=false, title='Pulp fiction', rating=8.9}</pre>

We can, of course, use Lambda expression instead of _Anonymous_ class as follows:

<pre class="lang:default decode:true">movies.sort((m1, m2) -&gt; {
    if(m1.getStarred() == m2.getStarred()){
        return 0;
    }
    return m1.getStarred() ? -1 : 1;
});</pre>

And we can also use again _Comparator.comparing():_

<pre class="lang:default decode:true">movies.sort(Comparator.comparing(Movie::getStarred, (star1, star2) -&gt; {
    if(star1 == star2){
         return 0;
    }
    return star1 ? -1 : 1;
}));</pre>

In the latest example _Comparator.comparing()_ takes as first parameter the function to extract the key to use for sorting and a Comparator as second parameter. This _Comparator_ uses the extracted keys for comparison, _star1_ and _star2 _are indeed _boolean _and represents _m1.getStarred()_ and _m2.getStarred() _respectively.

## 6. Sort a List with chain of Comparator

In the latest example, we want to have starred movie at the top and then sort by rating.

<pre class="lang:default decode:true ">List&lt;Movie&gt; movies = Arrays.asList(
        new Movie("Lord of the rings", 8.8, true),
        new Movie("Back to the future", 8.5, false),
        new Movie("Carlito's way", 7.9, true),
        new Movie("Pulp fiction", 8.9, false));

movies.sort(Comparator.comparing(Movie::getStarred)
                      .reversed()
                      .thenComparing(Comparator.comparing(Movie::getRating)
                      .reversed())
);

movies.forEach(System.out::println);
</pre>

And the output is:

<pre class="lang:diff decode:true ">Movie{starred=true, title='Lord of the rings', rating=8.8}
Movie{starred=true, title='Carlito's way', rating=7.9}
Movie{starred=false, title='Pulp fiction', rating=8.9}
Movie{starred=false, title='Back to the future', rating=8.5}</pre>

As you've seen we first sort by starred and then by rating &#8211; both reversed since we want the highest value and true first.