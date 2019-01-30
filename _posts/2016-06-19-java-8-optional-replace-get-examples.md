---
id: 78
title: 'Java 8 Optional &#8211; Replace your get() calls'
date: 2016-06-19T15:19:43+00:00
author: Mario Pio Gioiosa
layout: post
guid: http://reversecoding.net/?p=78
permalink: /java-8-optional-replace-get-examples/
site_layout:
  - full-width
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
Optional class were introduced in order to prevent _NullPointerException_,_ _but method _get()_ used to retrieve the value inside the Optional might still throw a _NoSuchElementException_.

Different name, same issue?

Calling _get() _without checking that value is actually present it's a bug. So we should always write something like that in order to use _get()._

<pre class="lang:java decode:true">Optional<String> myString = Optional.ofNullable(getNullableString());
   if(myString.isPresent()){
       doSomething(myString.get());
   }</pre>

### But are Optional really meant to be used in this way? No.

Writing block of i_sPresent/get_ it's not so different from writing a classic null check.

<pre class="lang:java decode:true">String myString = getNullableString();
   if(myString != null){
       doSomething(myString);
   }</pre>

Let's see how we can really benefit from Optional object.

## 1. Optional _orElse_ example

It returns the value if is present, or the other specified otherwise.

Let's see an example:

<pre class="lang:java decode:true">@Test
	public void orElse_whenNamePresent_ThenName(){
		Optional<String> petName = Optional.of("Bobby");
		
		assertEquals("Bobby", petName.orElse(""));
	}</pre>

<pre class="lang:java decode:true ">@Test
	public void orElse_whenNameNotPresent_ThenEmptyString(){
		Optional<String> petName = Optional.empty();
		
		assertEquals("", petName.orElse(""));
	}</pre>

As you can see we haven't called _get()_ and we've made the code easier and more readable compared to the _isPresent/get_ version:

<pre class="lang:java decode:true">@Test
	public void isPresentGet_whenNamePresent_ThenName(){
		Optional<String> petNameOptional = Optional.of("Bobby");
		
		String petName = "";
		if(petNameOptional.isPresent()){
			 petName = petNameOptional.get();
		} 
		
		assertEquals("Bobby", petName);
	}
	
	@Test
	public void isPresentGet_whenNameNotPresent_ThenEmptyString(){
		Optional<String> petNameOptional = Optional.empty();
		
		String petName = "";
		if(petNameOptional.isPresent()){
			 petName = petNameOptional.get();
		} 
		
		assertEquals("", petName);
	}</pre>

&nbsp;

## 2. Optional _orElseThrow_ example

It returns the value if is present, or throws the specified exception otherwise.

<pre class="lang:java decode:true">@Test
	public void elseOrThrow_whenNamePresent_ThenName(){
		Optional<String> petName = Optional.of("Bobby");
		
		assertEquals("Bobby", petName.orElse(""));
	}
	
	@Test(expected=IllegalArgumentException.class)
	public void elseOrThrow_whenNameNotPresent_ThenIllegalArgEx(){
		Optional<String> petName = Optional.empty();
		
		petName.orElseThrow(IllegalArgumentException::new);
	}</pre>

&nbsp;

## 3. Optional _filter_ example

_filter()_ is useful to specify other conditions on our object. It returns an Optional containing the value if is not empty and satisfy the specified predicate, an empty Optional otherwise.

In this example we want that the name not only is different from _null_ but also that is not empty or made of only empty spaces.

<pre class="lang:java decode:true">@Test
	public void filter_whenNameNotEmpty_thenName(){
		Optional<String> petNameOpt = Optional.of("Bobby");
		
		String petName = petNameOpt.filter(name -> !name.trim().isEmpty())
								   .orElseThrow(IllegalArgumentException::new);
		
		assertEquals("Bobby", petName);
	}</pre>

And those are the tests for the null and the empty name:

<pre class="lang:default decode:true ">@Test(expected=IllegalArgumentException.class)
	public void filter_whenNameNotPresent_thenIllegalArgEx(){
		Optional<String> petNameOpt = Optional.empty();
		
		petNameOpt.filter(name -> !name.trim().isEmpty())
				  .orElseThrow(IllegalArgumentException::new);
	}
	
	@Test(expected=IllegalArgumentException.class)
	public void filter_whenNameEmpty_thenIllegalArgEx(){
		Optional<String> petNameOpt = Optional.of(" ");
		
		petNameOpt.filter(name -> !name.trim().isEmpty())
				  .orElseThrow(IllegalArgumentException::new);
	}</pre>

## 4. Optional _ifPresent_ example

_[IfPresent](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html#ifPresent-java.util.function.Consumer-)_, that it's different from _isPresent_, accept a function, a Consumer, and executes it only if the value is present.

So instead of writing something like:

<pre class="lang:java decode:true ">if(optional.isPresent){
  doSomething(optional.get)
}</pre>

You can write:

<pre class="lang:java decode:true">optional.ifPresent(val -> doSomething(val))</pre>

or if you prefer:

<pre class="lang:java decode:true">optional.ifPresent(this::doSomething)</pre>

But let's have a look to a proper example.

We define a Pojo class, useful also for the following examples, that represents a Loyalty card.

<pre class="lang:java decode:true">public class LoyaltyCard {

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

}</pre>

We want to add 3 points to the loyalty card if the loyalty card is actually present.

_Node: In the following example we're going to use Mockito to mock LoyaltyCard class. Don't worry if you are not familiar with Mockito, I'll add some comments to the code._

<pre class="lang:java decode:true">@Test
	public void ifPresent_whenCardPresent_thenPointsAdded(){
		LoyaltyCard mockedCard = mock(LoyaltyCard.class);
		Optional<LoyaltyCard> loyaltyCard = Optional.of(mockedCard);
		
		loyaltyCard.ifPresent(c -> c.addPoints(3));
		
		//Verify addPoints method has been called 1 time and with input=3
		verify(mockedCard, times(1)).addPoints(3);
	}</pre>

&nbsp;

## 5. Optional _map_ example

_map()_ is a method used to transform an input in a different output. In this case, nothing changes except that the map operation will be executed only if the value is actually present, otherwise it returns an empty _Optional_.

In this example we want to retrieve the number of points of our loyalty card if we have it otherwise, the number of points will return 0.

<pre class="lang:default decode:true">@Test
	public void map_whenCardPresent_thenNumber(){
		LoyaltyCard mockedCard = mock(LoyaltyCard.class);
		when(mockedCard.getPoints()).thenReturn(3);
		
		Optional<LoyaltyCard> card = Optional.of(mockedCard);
		
		int point = card.map(LoyaltyCard::getPoints)
				   		.orElse(0);
		
		assertEquals(3, point);
	}</pre>

<pre class="lang:default decode:true ">@Test
	public void map_whenCardNotPresent_thenZero(){
		Optional<LoyaltyCard> card = Optional.empty();
		
		int point = card.map(LoyaltyCard::getPoints)
				   		.orElse(0);
		
		assertEquals(0, point);
	}</pre>

## 6. Optional _flatMap_ example

_flatMap()_ it's really similar to _map() _but when output is already an _Optional_ it doesn't wrap it with another _Optional_. So instead of having _Optional<Optional<T>>_ if will just return _Optional<T>_.

Let me clarify it using an example. Let's define a new class, called _Gift_.

<pre class="lang:java decode:true ">public class Gift {

	private String name;
	
// Constructor and getters

}</pre>

And let's define a new method to our LoyaltyCard class that returns an _Optional_ containing the last _Gift_ chosen. Since we are going to mock the result of this method, we don't really care about its implementation.

<pre class="lang:java decode:true">public Optional<Gift> getLastGift(){
		//whatever
		return Optional.empty();
	}</pre>

We can now create a mocked _Gift_ with name &#8220;Biography of Guybrush Threepwood&#8221;, put it into an _Optional_ and make _getLastGift _return it. So if we write:

<pre class="lang:default decode:true">card.map(LoyaltyCard::getLastGift)</pre>

The output will be an _Optional<Optional<Gift>> _that is not what we want, so flatMap will unwrap this double level and leave only an _Optional<Gift>._

<pre class="lang:default decode:true">@Test
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
	}</pre>

Writing this solution by using _isPresent/get_ would have meant using a nested if: one for check that card was present and another of checking the gift. Harder to read, easier to fail.

## 7. Optional _ifPresentOrElse ?_

Unfortunately, this is yet to come 🙂 [It will be available in Java 9](http://mail.openjdk.java.net/pipermail/core-libs-dev/2015-February/031223.html).

Until then we have to write something like:

<pre class="lang:java decode:true ">if(optional.isPresent()){
			doSomething(optional.get());
		} else {
			doSomethingElse();
		}</pre>

There are cases in which you are allowed to use _get()_ and _isPresent()_ but use them with a grain of salt.

**Resources:**

[JavaDoc](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)

[Short Tutorial By Example](http://www.javaspecialists.eu/archive/Issue238.html)