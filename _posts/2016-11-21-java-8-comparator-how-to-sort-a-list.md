---
id: 136
title: 'Java 8 Comparator - How to sort a List'
date: 2016-11-21T23:10:17+00:00
author: Mario Pio Gioiosa
layout: post
guid: http://reversecoding.net/?p=136
permalink: /java-8-comparator-how-to-sort-a-list/
categories:
  - Java 8
tags:
  - comparator
  - Java
  - Java 8
  - sort a list
  - sorting
---
In this article, we're going to see several examples on how to sort a `List` in Java 8.

## 1. Sort a List of String alphabetically

{% highlight java %}
List<String> cities = Arrays.asList(
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
//[Milan, New Delhi, San Francisco, Tokyo, london]
{% endhighlight %}

By purpose, we've written London with _'L'_ in low-case to better highlight difference between [Comparator.naturalOrder()](https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html#naturalOrder--) 
that returns a [Comparator](https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html) that sorts by placing capital letters first 
and [String.CASE_INSENSITIVE_ORDER](http://docs.oracle.com/javase/8/docs/api/java/lang/String.html#CASE_INSENSITIVE_ORDER) that returns a case-insensitive `Comparator`.

Basically, in Java 7 we were using `Collections.sort()` that was accepting a `List` and, eventually, a `Comparator` -  in Java 8 we have the new [List.sort()](http://docs.oracle.com/javase/8/docs/api/java/util/List.html#sort-java.util.Comparator-) that accepts a `Comparator`.

## 2. Sort a List of Integer

{% highlight java %}
List<Integer> numbers = Arrays.asList(6, 2, 1, 4, 9);
System.out.println(numbers); //[6, 2, 1, 4, 9]

numbers.sort(Comparator.naturalOrder());
System.out.println(numbers); //[1, 2, 4, 6, 9]
{% endhighlight %}


## 3. Sort a List by String field

Let's suppose we've our `Movie` class and we want to sort our `List` by title. We can use [Comparator.comparing()](https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html#comparing-java.util.function.Function-) and pass a function that extracts the field to use for sorting - title - in this example.

{% highlight java %}
List<Movie> movies = Arrays.asList(
        new Movie("Lord of the rings"),
        new Movie("Back to the future"),
        new Movie("Carlito's way"),
        new Movie("Pulp fiction"));

movies.sort(Comparator.comparing(Movie::getTitle));

movies.forEach(System.out::println);
{% endhighlight %}


The output will be:

{% highlight java %}
Movie{title='Back to the future'}
Movie{title='Carlito's way'}
Movie{title='Lord of the rings'}
Movie{title='Pulp fiction'}
{% endhighlight %}

As you've probably noticed we haven't passed any `Comparator` but the `List` is correctly sorted. That's because of the title - the extracted field - that is a `String` and `String` implements `Comparable` interface. 
If you peek at `Comparator.comparing()` implementation you will see that it calls _`compareTo`_ on the extracted key.

{% highlight java %}
return (Comparator<T> & Serializable)
            (c1, c2) -> keyExtractor.apply(c1).compareTo(keyExtractor.apply(c2));
{% endhighlight %}


## 4. Sort a List by double field

In a similar way, we can use [Comparator.comparingDouble()](https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html#comparingDouble-java.util.function.ToDoubleFunction-) for comparing `double` value. 
In the example, we want to order our `List` of `Movie` by rating, from the highest to the lowest.

{% highlight java %}
List<Movie> movies = Arrays.asList(
        new Movie("Lord of the rings", 8.8),
        new Movie("Back to the future", 8.5),
        new Movie("Carlito's way", 7.9),
        new Movie("Pulp fiction", 8.9));

movies.sort(Comparator.comparingDouble(Movie::getRating)
                      .reversed());

movies.forEach(System.out::println);
{% endhighlight %}

We used [reversed](https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html#reversed--) function on the `Comparator` in order to invert default natural-order that is from lowest to highest. 
`Comparator.comparingDouble()` uses `Double.compare()` under the hood.

If you need to compare `int` or `long` you can use `comparingInt()` and `comparingLong()` respectively.

## 5. Sort a List with custom Comparator

In the previous examples we haven't specified any Comparator since it wasn't necessary but let's see an example in which we define our own _Comparator_. 
Our _Movie_ class has a new field - starred - set using the third constructor parameter. In the example, we want to sort the list so that we have starred movie at the top of the _List_. 

{% highlight java %}
List<Movie> movies = Arrays.asList(
        new Movie("Lord of the rings", 8.8, true),
        new Movie("Back to the future", 8.5, false),
        new Movie("Carlito's way", 7.9, true),
        new Movie("Pulp fiction", 8.9, false));

movies.sort(new Comparator<Movie>() {
    @Override
    public int compare(Movie m1, Movie m2) {
        if(m1.getStarred() == m2.getStarred()){
            return 0;
        }
        return m1.getStarred() ? -1 : 1;
     }
});

movies.forEach(System.out::println);
{% endhighlight %}


The result will be:

{% highlight java %}
Movie{starred=true, title='Lord of the rings', rating=8.8}
Movie{starred=true, title='Carlito's way', rating=7.9}
Movie{starred=false, title='Back to the future', rating=8.5}
Movie{starred=false, title='Pulp fiction', rating=8.9}
{% endhighlight %}


We can, of course, use Lambda expression instead of _Anonymous_ class as follows:

{% highlight java %}
movies.sort((m1, m2) -> {
    if(m1.getStarred() == m2.getStarred()){
        return 0;
    }
    return m1.getStarred() ? -1 : 1;
});
{% endhighlight %}


And we can also use again `Comparator.comparing()`:

{% highlight java %}
movies.sort(Comparator.comparing(Movie::getStarred, (star1, star2) -> {
    if(star1 == star2){
         return 0;
    }
    return star1 ? -1 : 1;
}));
{% endhighlight %}


In the latest example _Comparator.comparing()_ takes as first parameter the function to extract the key to use for sorting and a _Comparator_ as second parameter. 
This _Comparator_ uses the extracted keys for comparison, _star1_ and _star2_ are indeed `boolean` and represents _m1.getStarred()_ and _m2.getStarred()_ respectively.

## 6. Sort a List with chain of Comparator

In the latest example, we want to have starred movie at the top and then sort by rating.

{% highlight java %}
List<Movie> movies = Arrays.asList(
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

{% endhighlight %}


And the output is:

{% highlight java %}
Movie{starred=true, title='Lord of the rings', rating=8.8}
Movie{starred=true, title='Carlito's way', rating=7.9}
Movie{starred=false, title='Pulp fiction', rating=8.9}
Movie{starred=false, title='Back to the future', rating=8.5}
{% endhighlight %}


As you've seen we first sort by starred and then by rating - both reversed since we want the highest value and true first.