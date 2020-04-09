---
title: 'Java 8 - Convert List to String comma separated'
date: 2016-05-22T10:30:12+00:00
author: Mario Pio Gioiosa
layout: post
permalink: /java-8-convert-list-string-comma/
description: Convert a List of String in a String with all the values of the List comma separated, using Java 8, is really straightforward.
tags: [java, stream]
---
Convert a `List<String>` in a `String` with all the values of the `List` comma separated using Java 8 is really straightforward.
Let's have a look on how to do that.

### Java 8 (or above)
We simply can write _String.join(..)_, pass a delimiter and an [Iterable](https://docs.oracle.com/javase/8/docs/api/java/lang/Iterable.html), then the new [StringJoiner](https://docs.oracle.com/javase/8/docs/api/java/util/StringJoiner.html) will do the rest:

{% highlight java %}
List<String> cities = Arrays.asList("Milan",
                                    "London",
                                    "New York",
                                    "San Francisco");

String citiesCommaSeparated = String.join(",", cities);

System.out.println(citiesCommaSeparated);
// prints 'Milan,London,New York,San Francisco' to STDOUT.
{% endhighlight %}

In case we are working with a _[stream](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html)_ we can write as follow and still have the same result:

{% highlight java %}
String citiesCommaSeparated = cities.stream()
                                    .collect(Collectors.joining(","));

System.out.println(citiesCommaSeparated);

//Output: Milan,London,New York,San Francisco
{% endhighlight %}


Note: you can statically *import java.util.stream.Collectors.joining* if you prefer just typing *joining(",")*.

### Java 7
For the old times' sake, let's have a look at Java 7 implementation.

{% highlight java %}
private static final String SEPARATOR = ",";

public static void main(String[] args) {
  List<String> cities = Arrays.asList(
                                "Milan",
                                "London",
                                "New York",
                                "San Francisco");

  StringBuilder csvBuilder = new StringBuilder();

  for(String city : cities){
    csvBuilder.append(city);
    csvBuilder.append(SEPARATOR);
  }

  String csv = csvBuilder.toString();
  System.out.println(csv);
  //OUTPUT: Milan,London,New York,San Francisco,

  //Remove last comma
  csv = csv.substring(0, csv.length() - SEPARATOR.length());

  System.out.println(csv);
  //OUTPUT: Milan,London,New York,San Francisco
{% endhighlight %}


As you can see it's much more verbose and easier to make mistakes like forgetting to remove the last comma. You can implement this in several ways - for example by moving the logic that removes the last comma inside the *for* loop - but none will be so explicative and immediate to understand as the declarative solution expressed in Java 8.

The focus should be on what you want to do - joining a `List` of `String` - not on how.

### Java 8: Manipulate String before joining
Before joining you can manipulate your `String` as you prefer by using *map()* or cutting some `String` out by using *filter()*.  I'll cover those topics in further articles. Meanwhile, this a straightforward example on how to transform the whole String in upper-case before joining them.

#### Java 8: From List to upper-case String comma separated
{% highlight java %}
String citiesCommaSeparated = cities.stream()
                                    .map(String::toUpperCase)
                                    .collect(Collectors.joining(","));

//Output: MILAN,LONDON,NEW YORK,SAN FRANCISCO
{% endhighlight %}

If you want to find out more about stream, I strongly suggest [this](https://vimeo.com/124034512) cool video of [Venkat Subramaniam](https://twitter.com/venkat_s).

### Let's play

The best way to learn it's playing! Copy this class with all the implementations discussed and play with that. There is already a small test for each of them 🙂

{% highlight java %}
package net.reversecoding.examples;

import static java.util.stream.Collectors.joining;
import static org.junit.Assert.assertEquals;

import java.util.Arrays;
import java.util.List;

import org.junit.Test;

public class CsvUtil {
    private static final String SEPARATOR = ",";

    public static String toCsv(List<String> listToConvert){
        return String.join(SEPARATOR, listToConvert);
    }

    @Test
    public void toCsv_csvFromListOfString(){
        List<String> cities = Arrays.asList(
                "Milan", "London", "New York", "San Francisco");

        String expected = "Milan,London,New York,San Francisco";

        assertEquals(expected, toCsv(cities));
    }


    public static String toCsvStream(List<String> listToConvert){
        return listToConvert.stream()
                    .collect(joining(SEPARATOR));
    }

    @Test
    public void toCsvStream_csvFromListOfString(){
        List<String> cities = Arrays.asList(
                "Milan", "London", "New York", "San Francisco");

        String expected = "Milan,London,New York,San Francisco";

        assertEquals(expected, toCsv(cities));
    }

    public static String toCsvJava7(List<String> listToConvert){
        StringBuilder csvBuilder = new StringBuilder();

        for(String s : listToConvert){
            csvBuilder.append(s);
            csvBuilder.append(SEPARATOR);
        }

        String csv = csvBuilder.toString();

        //Remove last separator
        if(csv.endsWith(SEPARATOR)){
            csv = csv.substring(0, csv.length() - SEPARATOR.length());
        }

        return csv;
    }

    @Test
    public void toCsvJava7_csvFromListOfString(){
        List<String> cities = Arrays.asList(
                "Milan", "London", "New York", "San Francisco");

        String expected = "Milan,London,New York,San Francisco";

        assertEquals(expected, toCsvJava7(cities));
    }

    public static String toUpperCaseCsv(List<String> listToConvert){
        return listToConvert.stream()
                    .map(String::toUpperCase)
                    .collect(joining(SEPARATOR));
    }

    @Test
    public void toUpperCaseCsv_upperCaseCsvFromListOfString(){
        List<String> cities = Arrays.asList(
                "Milan", "London", "New York", "San Francisco");

        String expected = "MILAN,LONDON,NEW YORK,SAN FRANCISCO";

        assertEquals(expected, toUpperCaseCsv(cities));
    }
}
{% endhighlight %}