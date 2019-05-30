---
id: 78
title: 'Java 8 Optional - Replace your get() calls'
date: 2016-06-19T15:19:43+00:00
author: Mario Pio Gioiosa
layout: post
guid: http://reversecoding.net/?p=78
permalink: /java-8-optional-replace-get-examples/
tags: [java, optional]
---
Optional class were introduced in order to prevent `NullPointerException`, but method `get()` used to retrieve the value inside the `Optional` might still throw a `NoSuchElementException`.

Different name, same issue?

Calling `get()` without checking that the value is actually present it's a problem. So we should always write something like this in order to use `get()`.

{% highlight java %}
Optional<String> myString = Optional.ofNullable(nullableString());
   if(myString.isPresent()){
       doSomething(myString.get());
   }
{% endhighlight %}

#### *But are Optional really meant to be used in this way? Actually, no.*

Writing block of `isPresent/get` is not so different from writing a classic null check.

{% highlight java %}
String myString = nullableString();
   if(myString != null){
       doSomething(myString);
   }
{% endhighlight %}

How can we really benefit from `Optional` object?

## 1. Optional _orElse_ example
It returns the value if is present, or the other specified otherwise.

Let's see an example:

{% highlight java %}
@Test
public void namePresent(){
  Optional<String> petName = Optional.of("Bobby");

  assertEquals("Bobby", petName.orElse(""));
}
{% endhighlight %}


{% highlight java %}
@Test
public void orElse(){
  Optional<String> petName = Optional.empty();

  assertEquals("", petName.orElse(""));
}
{% endhighlight %}

As you can see the code is much more readable and correct compared to the _isPresent/get_ version:

{% highlight java %}
String petName = "";
if(petNameOptional.isPresent()){
    petName = petNameOptional.get();
}
{% endhighlight %}

## 2. Optional _orElseThrow_ example
It returns the value if it is present, or throws the specified exception otherwise.

{% highlight java %}
@Test
public void namePresent(){
  Optional<String> petName = Optional.of("Bobby");

  assertEquals("Bobby", petName.orElseThrow(IllegalArgumentException::new));
}

@Test(expected=IllegalArgumentException.class)
public void throwException(){
  Optional<String> petName = Optional.empty();

  petName.orElseThrow(IllegalArgumentException::new);
}
{% endhighlight %}

## 3. Optional _filter_ example
_filter()_ is useful to specify other conditions on our object. 
It returns an `Optional` containing the value if it is not empty and satisfies the specified predicate, an `Optional.empty()` otherwise.

In this example we want that the name length should be greater than 3.

{% highlight java %}
@Test
public void valueNotFilteredOut()
{
  Optional<String> petName = Optional.of("Bobby")
                                     .filter(name -> name.length() > 3);
  assertEquals(Optional.of("Bobby"), petName);
}

@Test
public void valueFilteredOut()
{
  Optional<String> petName = Optional.of("Bob")
                                     .filter(name -> name.length() > 3);
  assertEquals(Optional.empty(), petName);
}
{% endhighlight %}

## 4. Optional _map_ example
_map()_ is a method used to apply a transformation to the content of the `Optional` if it's present. 

In this example we want to transform the name of the pet from String to Int by applying `String.length()` function.

{% highlight java %}
@Test
public void transformWhenValueIsPresent()
{
  Optional<Integer> nameLength = Optional.of("Bobby")
                                         .map(String::length);
  assertEquals(Optional.of(5), nameLength);
}

@Test
public void mapNotExecutedWhenEmpty()
{
  Optional<Integer> nameLength = Optional.<String>empty()
                                         .map(String::length);
  assertEquals(Optional.empty(), nameLength);
}
{% endhighlight %}

*What's interesting with the `map` method is that we are able to declare sequentially our operation without worrying 
if the value is present or not. We can focus on the success case, we don't need to branch our code, the scenario in
which the value is not present is automatically handled by the abstraction `Optional` that we are using.*

## 5. Optional _flatMap_ example

_flatMap()_ it's similar to _map()_ but should be used when the transformation function returns an `Optional` of some `<T>`.
_flatMap()_ executes the transformation function like the _map()_ but instead of returning `Optional<Optional<T>>` if will just return `Optional<T>`.
Basically it flattens the `Optional`.

Let's clarify it with an example. We define a function that returns an `Optional`, let's say a function
that returns the last gift received by the pet.

{% highlight java %}
private Optional<String> lastGiftReceivedBy(String petName) = ???
{% endhighlight %}

If we call our _lastGiftReceivedBy_ function in a _map()_ we will have an `Optional<Optional<String>>`.
{% highlight java %}
Optional<Optional<String>> lastGiftReceived = Optional.of("Bobby")
                                                      .map(petName -> lastGiftReceivedBy(petName));
{% endhighlight %}

In order to flatten them we can use _flatMap()_

{% highlight java %}
Optional<String> lastGiftReceived = Optional.of("Bobby")
                                            .flatMap(petName -> lastGiftReceivedBy(petName));

{% endhighlight %}

Writing this solution by using _isPresent/get_ would have meant using a nested if: one for the first
`Optional`, one for the other. 

## 6. Optional _ifPresent_ example

_[IfPresent](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html#ifPresent-java.util.function.Consumer-)_, that it's different from _isPresent_, accept a function, a `Consumer`, and executes it only if the value is present. 

Basically _ifPresent()_ should be used for function that does side-effect (returning a void).

Instead of writing something like:

{% highlight java %}
if(optional.isPresent){
  doSomething(optional.get)
}
{% endhighlight %}

We can write:

{% highlight java %}
optional.ifPresent(val -> doSomething(val))
{% endhighlight %}

Let's say we want to do a _System.out.println()_ of the name:

{% highlight java %}
Optional.of("Bobby")
        .ifPresent(name -> System.out.println(name));
{% endhighlight %}

**Resources:**

[JavaDoc](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)

[Short Tutorial By Example](http://www.javaspecialists.eu/archive/Issue238.html)