---
title: 'Java 8 Stream - From List to Map'
date: 2016-06-04T15:34:33+00:00
author: Mario Pio Gioiosa
layout: post
permalink: /java-8-list-to-map/
description: An example to convert a `List<?>` to a `Map<K,V>` using Java 8 Stream
tags: [java, stream]
---
An example to convert a `List<?>` to a `Map<K,V>` using Java 8 _[Stream](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html)_.

### Java 8 - Collectors.toMap()
Let's define a Pojo class:

{% highlight java %}
public class Person {

  private String email;
  private String name;
  private int age;

  public Person(String email, String name, int age) {
    this.email = email;
    this.name = name;
    this.age = age;
  }

  //Getters
  //toString
}
{% endhighlight %}

In the first example, we convert a `List<Person>` in a `Map<String, Person>` that has the email as key and the object itself as value.

{% highlight java %}
List<Person> people = Arrays.asList(
     new Person("mario@reversecoding.net", "Mario", 27),
     new Person("luigi@reversecoding.net", "Luigi", 30),
     new Person("steve@reversecoding.net", "Steve", 20)
 );

 Map<String, Person> mapEmailPerson = people.stream()
                .collect(Collectors.toMap(Person::getEmail, Function.identity()));
{% endhighlight %}

or with the lambda instead of the method reference:
{% highlight java %}
Map<String, Person> mapEmailPerson = people.stream()
               .collect(Collectors.toMap(person -> person.getEmail(), 
                                         person -> person));
{% endhighlight %}

Basically `mapEmailPerson` is a `Map<Strig, Person>` in which the key is the *email* and the value is the *Person* instance with that specific email.

### Let's break this out
First of all, we create a `Stream` of `Person` from the `List<Person>` defined.

Then we collect this stream in a `Map`. Java 8 helps us to define the needed `Collector` by providing us the method: [Collectors.toMap()](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html#toMap-java.util.function.Function-java.util.function.Function-).

`Collectors.toMap()` takes two functions - one for mapping the key and one for the value - and returns a `Collector` that accumulates elements into a `Map`.

We have chosen the email as key, so the first parameter of the *toMap* is a function that given the input - a `Person` - returns its email:

{% highlight java %}
//Given a person, we get his email
person -> person.getEmail()

//Or using method reference
Person::getEmail()
{% endhighlight %}

The value is the object itself, so the second parameter of the *toMap* is simply an identity function:

{% highlight java %}
//Given a person, we want as value the person itself
person -> person

//Or
Function.identity()
{% endhighlight %}

These are basically the two parameters for `toMap`.

### Another example
Given a `List<Person>` we want to create a `Map<String, Integer>` that contains the name as key and the age as value.

We just need to change the two parameters of the `Collectors.toMap`, by specifying:

{% highlight java %}
//Name as key
person -> person.getName()

//Age as value
person -> person.getAge();
{% endhighlight %}

So the code will look like:

{% highlight java %}
List<Person> people = Arrays.asList(
    new Person("mario@reversecoding.net", "Mario", 27),
    new Person("luigi@reversecoding.net", "Luigi", 30),
    new Person("steve@reversecoding.net", "Steve", 20)
);

Map<String, Integer> mapNameAge = people.stream()
          .collect(Collectors.toMap(person -> person.getName(), 
                                    person -> person.getAge()));

System.out.println(mapNameAge);
//{Steve=20, Luigi=30, Mario=27}
{% endhighlight %}

If you are a good observer you may have noticed that the order has not been respected. That's because the default implementation used by _toMap()_ is the
[_HashMap_](https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html) that does not guarantee that the insert order will be respect.

### Java 8 - Collectors.toMap with a LinkedHashMap
If we want to preserve the order we should use a _[LinkedHashMap](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashMap.html)_ instead of the HashMap.

Let's try the previous example by passing a `LinkedHashMap` to `Collectors.toMap()`

{% highlight java %}
Map<String, Integer> mapNameAge = people.stream()
      .collect(Collectors.toMap(
          Person::getName,
          Person::getAge,
          (u,v) -> { throw new IllegalStateException(String.format("Duplicate key %s", u)); },
          LinkedHashMap::new
          ));

System.out.println(mapNameAge);
//{Mario=27, Luigi=30, Steve=20}
{% endhighlight %}

We are using the definition of [toMap that takes four parameters](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html#toMap-java.util.function.Function-java.util.function.Function-java.util.function.BinaryOperator-java.util.function.Supplier-):

  * `keyMapper` - a mapping function to produce the keys
  * `valueMapper` - a mapping function to produce the values
  * `mergeFunction` - a merge function used to resolve collisions between values associated with the same key
  * `mapSupplier` - a function which returns a new, empty `Map` into which the results will be inserted

We've already discussed the first two parameters.

In case of a collision we just want to throw an exception, so as third parameter we define exactly this behaviour. In the example, we used the same implementation of the static method _throwingMerger _defined in the java.util.stream.Collectors class.

The fourth parameter it's the one in which we define a function that returns our `LinkedHashMap`.