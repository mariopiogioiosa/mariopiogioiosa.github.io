---
id: 251
title: SOLID principles explained by Sandro Mancuso
date: 2017-08-05T10:54:22+00:00
author: Mario Pio Gioiosa
layout: post
guid: https://reversecoding.net/?p=251
permalink: /solid-principles-explained-sandro-mancuso/
categories:
  - OO
tags:
  - Clean code
  - Devoxx
  - FP
  - Functional programming
  - Object oriented
  - Object oriented principles
  - Object oriented programming
  - OO
  - OO principles
  - Sandro mancuso
  - Software Craftsmanship
  - SOLID
  - SOLID principles
---
### SOLID principles

Have you ever heard **SOLID** principles?

**SOLID** stands for:

  * **S**ingle responsibility principle
  * **O**pen/closed principle
  * **L**iskov substitution principle
  * **I**nterface segregation principle
  * **D**ependency inversion principle

### Easy to understand, easier to violate

Those principles are not difficult to understand when you initially study them, however, stick to them in your daily job is not trivial at all. Let's make an example on the first one &#8211; Single Responsibility Principle &#8211; the easiest and probably the most violated one.

Let's imagine you have a function named _calculateTotalPrice(..) _and it's long, I don't know, let's say 50 lines of code. **Is the single responsibility principle respected?**

Maybe yes, is doing one thing: calculate the total price of the cart. However, when we look at the code we see something like that:

<pre class="lang:default decode:true">Price calculateTotalPrice(){
  //15 lines to multiply price of the single product by ordered quantity 
  //10 to sum all the entries of the cart
  //15 to apply discounts
  //10 lines to apply taxes
}</pre>

Are you still convinced the method is following Single Responsibility Principle? Clearly not.

It does **five** things:

  * Coordinate total price calculation by applying sequentially **four** operations
  * Calculate single entry price
  * Sum all entries price
  * Apply discount
  * Apply taxes

If one of those **five** things changes, we have to touch this method &#8211; clearly it has more than one responsibility.

**It's really easy to violate them without even realizing it.**

### &#8220;Functional is cool, but do you know OO&#8221; by Sandro Mancuso

So today we will not speak about [Java 8](https://reversecoding.net/category/java-8/) or [Spring MVC](https://reversecoding.net/category/spring-mvc/) but Objected Oriented principles.

We start by suggesting this talk by [Sandro Mancuso](https://twitter.com/sandromancuso).

&nbsp;

<span class="embed-youtube" style="text-align:center; display: block;"></span>