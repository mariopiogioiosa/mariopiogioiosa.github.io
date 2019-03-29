---
id: 78
title: 'Java 8 Optional - Replace your get() calls'
date: 2016-06-19T15:19:43+00:00
author: Mario Pio Gioiosa
layout: post
guid: http://reversecoding.net/?p=78
permalink: /java-8-optional-replace-get-examples/
categories:
  - Java 8
tags:
  - example
  - filter
  - flatMap
  - Java 8
  - map
  - Optional
  - orElse
  - orElseThrow
---
Optional class were introduced in order to prevent `NullPointerException`, but method `get()` used to retrieve the value inside the `Optional` might still throw a `NoSuchElementException`.

Different name, same issue?

Calling `get()` without checking that the value is actually present it's a problem. So we should always write something like this in order to use `get()`.

{% highlight java %}
Optional<String> myString = Optional.ofNullable(getNullableString());
   if(myString.isPresent()){
       doSomething(myString.get());
   }
{% endhighlight %}

#### *But are Optional really meant to be used in this way? No.*

Writing block of `isPresent/get` is not so different from writing a classic null check.

{% highlight java %}
String myString = getNullableString();
   if(myString != null){
       doSomething(myString);
   }
{% endhighlight %}

How can we benefit from `Optional` object?

## 1. Optional _orElse_ example
It returns the value if is present, or the other specified otherwise.

Let's see an example:

{% highlight java %}
@Test
public void orElse_whenNamePresent_ThenName(){
  Optional<String> petName = Optional.of("Bobby");

  assertEquals("Bobby", petName.orElse(""));
}
{% endhighlight %}


{% highlight java %}
@Test
public void orElse_whenNameNotPresent_ThenEmptyString(){
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
public void elseOrThrow_whenNamePresent_ThenName(){
  Optional<String> petName = Optional.of("Bobby");

  assertEquals("Bobby", petName.orElseThrow(IllegalArgumentException::new));
}

@Test(expected=IllegalArgumentException.class)
public void elseOrThrow_whenNameNotPresent_ThenIllegalArgEx(){
  Optional<String> petName = Optional.empty();

  petName.orElseThrow(IllegalArgumentException::new);
}
{% endhighlight %}

## 3. Optional _filter_ example
_filter()_ is useful to specify other conditions on our object. It returns an `Optional` containing the value if it is not empty and satisfies the specified predicate, an `Optional.empty()` otherwise.

In this example we want that the name should not be empty or with only empty spaces.

{% highlight java %}
@Test
public void filter_whenNameNotEmpty_thenName(){
  Optional<String> petNameOpt = Optional.of("Bobby");

  String petName = petNameOpt.filter(name -> !name.trim().isEmpty())
                 .orElseThrow(IllegalArgumentException::new);

  assertEquals("Bobby", petName);
}


@Test(expected=IllegalArgumentException.class)
public void filter_whenNameEmpty_thenIllegalArgEx(){
  Optional<String> petNameOpt = Optional.of(" ");

  petNameOpt.filter(name -> !name.trim().isEmpty())
        .orElseThrow(IllegalArgumentException::new);
}
{% endhighlight %}

## 4. Optional _map_ example
_map()_ is a method used to apply a transformation to the content of the `Optional` if it's present. 

But let's have a look to a proper example.
We define a Pojo class, useful also for the following examples, that represents a loyalty card.

{% highlight java %}
public class LoyaltyCard {

  private String cardNumber;
  private int points;

  public LoyaltyCard(String cardNumber, int points){
    this.cardNumber = cardNumber;
    this.points = points;
  }

  public int addPoints(int pointToAdd){
    return points += pointToAdd;
  }

  //Getters
}
{% endhighlight %}

We want to retrieve the number of points of our loyalty card if we have it otherwise, the number of points will return 0.

_Node: In the following example we're going to use Mockito to mock LoyaltyCard class. Don't worry if you are not familiar with Mockito, we'll add some comments to the code._

{% highlight java %}
@Test
public void map_whenCardPresent_thenNumber(){
    LoyaltyCard mockedCard = mock(LoyaltyCard.class);
    when(mockedCard.getPoints()).thenReturn(3);

    Optional<LoyaltyCard> card = Optional.of(mockedCard);

    int point = card.map(LoyaltyCard::getPoints)
               .orElse(0);

    assertEquals(3, point);
}
{% endhighlight %}

{% highlight java %}
@Test
public void map_whenCardNotPresent_thenZero(){
    Optional<LoyaltyCard> card = Optional.empty();

    int point = card.map(LoyaltyCard::getPoints)
               .orElse(0);

    assertEquals(0, point);
}
{% endhighlight %}

## 5. Optional _flatMap_ example

_flatMap()_ it's really similar to _map()_ but when output is already an `Optional` it doesn't wrap it with another _Optional_. So instead of having `Optional<Optional<T>>` if will just return `Optional<T>`.

Let me clarify it using an example. Let's define a new class, called _Gift_.

{% highlight java %}

public class Gift {

  private String name;

// Constructor and getters

}
{% endhighlight %}

And let's define a new method to our LoyaltyCard class that returns an _Optional_ containing the last _Gift_ chosen. Since we are going to mock the result of this method, we don't really care about its implementation.

{% highlight java %}
public Optional<Gift> getLastGift(){
    //whatever
    return Optional.empty();
  }
{% endhighlight %}

We can now create a mocked _Gift_ with name "Biography of Guybrush Threepwood", put it into an _Optional_ and make _getLastGift _return it. So if we write:

{% highlight java %}
card.map(LoyaltyCard::getLastGift)
{% endhighlight %}

The output will be an `Optional<Optional<Gift>>` that is not what we want, so flatMap will unwrap this double level and leave only an `Optional<Gift>`.

{% highlight java %}
@Test
public void flatMap_whenCardAndLastGiftPresent_thenName(){
    Gift mockedGift = mock(Gift.class);
    when(mockedGift.getName()).thenReturn("Biography of Guybrush Threepwood");

    LoyaltyCard mockedCard = mock(LoyaltyCard.class);
    when(mockedCard.getLastGift()).thenReturn(Optional.of(mockedGift));
    Optional<LoyaltyCard> card = Optional.of(mockedCard);

    String giftName = card.flatMap(LoyaltyCard::getLastGift)
                .map(Gift::getName)
                .orElse("");

    assertEquals("Biography of Guybrush Threepwood", giftName);
}
{% endhighlight %}

Writing this solution by using _isPresent/get_ would have meant using a nested if: one for check that card was present and another of checking the gift. Harder to read, easier to fail.

## 6. Optional _ifPresent_ example

_[IfPresent](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html#ifPresent-java.util.function.Consumer-)_, that it's different from _isPresent_, accept a function, a `Consumer`, and executes it only if the value is present. 

So instead of writing something like:

{% highlight java %}
if(optional.isPresent){
  doSomething(optional.get)
}
{% endhighlight %}

You can write:

{% highlight java %}
optional.ifPresent(val -> doSomething(val))
{% endhighlight %}

or if you prefer:

{% highlight java %}
optional.ifPresent(this::doSomething)
{% endhighlight %}

We want to add 3 points to the loyalty card if the loyalty card is actually present.

{% highlight java %}
@Test
public void ifPresent_whenCardPresent_thenPointsAdded(){
  LoyaltyCard mockedCard = mock(LoyaltyCard.class);
  Optional<LoyaltyCard> loyaltyCard = Optional.of(mockedCard);

  loyaltyCard.ifPresent(c -> c.addPoints(3));

  //Verify addPoints method has been called 1 time and with input=3
  verify(mockedCard, times(1)).addPoints(3);
}
{% endhighlight %}


**Resources:**

[JavaDoc](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)

[Short Tutorial By Example](http://www.javaspecialists.eu/archive/Issue238.html)