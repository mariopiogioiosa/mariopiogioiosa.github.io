---
id: 191
title: 'Spring MVC @Requestparam &#8211; Binding request parameters'
date: 2017-02-26T23:54:19+00:00
author: Mario Pio Gioiosa
layout: post
guid: https://reversecoding.net/?p=191
permalink: /spring-mvc-requestparam-binding-request-parameters/
hefo_before:
  - "0"
hefo_after:
  - "0"
categories:
  - Spring MVC
tags:
  - @RequestParam
  - binding
  - Java
  - query string
  - request parameters
  - Spring
  - spring mvc
---
In this article, we are going to see several examples on how to get request parameters with Spring MVC, how to bind them to different objects, how to use _`@RequestParam`_ annotation and when the annotation is not needed.

## @RequestParam examples

_`@RequestParam`_ is an annotation which indicates that a method parameter should be bound to a web request parameter.

### 1. Biding method parameter with web request parameter

Let&#8217;s say we have our _/books_ URL that requires a mandatory parameter named _category:_

<pre class="lang:default decode:true">/books?category=java</pre>

we just need to add _`@RequestParam("parameterName")`_ annotation before the method parameter that we will use for biding the web request parameter.
  
Let&#8217;s see an example:

<pre class="lang:default decode:true">@RequestMapping("/books")
public String books(@RequestParam("category") String cat, Model model){
    //business logic...
    model.addAttribute("category", cat);
    return "books.jsp";
}</pre>

<span id="default-test"></span>In the snippet above we&#8217;ve captured value of request parameter &#8220;category&#8221; into _`String cat`_ method argument and we put its value into a _`Model`_ object so that we can easily test it.

<pre class="lang:default decode:true">@Test
public void getCategoryParam() throws Exception {
    this.mockMvc.perform(get("/books")
            .param("category", "java")
    )
            .andExpect(status().isOk())
            .andExpect(model().attribute("category", is("java")))
            .andDo(print());
}</pre>

In the test above we are using [Spring MVC](http://docs.spring.io/spring-security/site/docs/current/reference/html/test-mockmvc.html) test library to perform an HTTP GET request to **_/books_** with a parameter &#8211; _category_ &#8211; which has value &#8220;_java&#8221;._ Then we verify that the response status is OK (code: 200) and that the model contains an attribute named _category_ and that its value is &#8220;java&#8221;.

_For this test we&#8217;ve quickly setup a Spring Boot project, you can find the pom.xml [at the end of this article](#pom-xml)._

**Please note** that _category _is considered a **mandatory** parameter if we don&#8217;t pass _category_ parameter in our call as in the test below we receive a 400: _`BadRequestException`_.

<pre class="lang:default decode:true">@Test
public void whenNoParam_thenBadRequest() throws Exception {
    this.mockMvc.perform(get("/books"))
            .andExpect(status().isBadRequest());
}</pre>

### 2. Request parameter match method parameter name

If both variable and request parameter name matches we don&#8217;t need to specify the parameter name in the _`@RequestParam`_ annotation.

<pre class="lang:default decode:true ">@RequestMapping("/books")
public String books(@RequestParam String category, Model model){
    model.addAttribute("category", category);
    return "books.jsp";
}</pre>

In the example above, since** **both variable and request parameter name is &#8216;category&#8217;, binding is performed automatically.

### 3. Auto type conversion

If the request parameter is not a _`String`_ but &#8211; for example &#8211; a number we can bind it to the corresponding type. Let&#8217;s say we have a call like this:

<pre class="lang:default decode:true ">/books?rate=5&maxprice=150.00</pre>

In order to bind _rate_ and _maxprice _respectively_ _with an _`int`_ and a _`BigDecimal`_, we just need to specify the target type after the annotation.

<pre class="lang:default decode:true">@RequestMapping("/books")
public String books(@RequestParam("rate") int rate,
                         @RequestParam("maxprice") BigDecimal maxprice, Model model){
    model.addAttribute("rate", rate);
    model.addAttribute("maxprice", maxprice);
    return "books.jsp";
}</pre>

<pre class="lang:default decode:true">@Test
public void whenNumber_thenAutoType() throws Exception {
    this.mockMvc.perform(get("/books")
            .param("rate", "5")
            .param("maxprice", "150.00")
    )
            .andExpect(status().isOk())
            .andExpect(model().attribute("rate", is(5)))
            .andExpect(model().attribute("maxprice", is(new BigDecimal("150.00"))));
}</pre>

Similarly, if we want to pass a date like:

<pre class="lang:default decode:true ">/books?from=2011-01-01</pre>

We need to bind it with respective object:

<pre class="lang:default decode:true">@RequestMapping("/books")
public String books(@DateTimeFormat(iso = DateTimeFormat.ISO.DATE)
                    @RequestParam("from") LocalDate from, Model model){
    model.addAttribute("from", from);
    return "books.jsp";
}</pre>

<pre class="lang:default decode:true">@Test
public void whenDate_thenAutoType() throws Exception {
    this.mockMvc.perform(get("/books")
            .param("from", "2011-01-01")
    )
            .andExpect(status().isOk())
            .andExpect(model().attribute("from", is(LocalDate.of(2011,01,01))));
}</pre>

Since dates have different formats, it&#8217;s necessary to specify which one should be used in order to parse the value parameter correctly. In our example, we&#8217;ve used [_`@DateTimeFormat`_](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/format/annotation/DateTimeFormat.html)  to specify iso _`DateTimeFormat.ISO.DATE`_ that is the most common ISO: `yyyy-MM-dd`.

All simple types such as _`int`_,_`long`_, _`Date`_, etc. are supported.

The conversion process can be configured and customized according to your needs using _[`WebBinder`](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/mvc.html#mvc-ann-webdatabinder)_ or by registering _[`Formatters`](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/validation.html#format). _

### 4. @RequestParam with not mandatory parameters

When we add_`@RequestParam`_ annotation, as we&#8217;ve seen, by default we are assuming that request parameter is mandatory. In case we want to specify that is not, we can just add _`required=false`_.

<pre class="lang:default decode:true">@RequestMapping("/books")
public String books(@RequestParam(name="category", required = false) String category, 
    Model model){
    model.addAttribute("category", category);
    return "books.jsp";
}</pre>

In this case, if we call just _/books _without parameter, we won&#8217;t receive a 400 as before. _`String category`_ in this case will remain _null_.

<pre class="lang:default decode:true">@Test
public void whenParameterNotSent_thenStatusOk() throws Exception {
    this.mockMvc.perform(get("/books"))
            .andExpect(status().isOk())
            .andExpect(model().attribute("category", is(nullValue())));
}</pre>

### **5. @RequestParam with Default value **

It&#8217;s also possible to specify a default value that will be applied if the parameter is not sent. Let&#8217;s say our default value is &#8220;fantasy&#8221; we can add _`defaultValue = "fantasy"`_ in the annotation as follows:

<pre class="lang:default decode:true">@RequestMapping("/books")
public String books(@RequestParam(name="category", defaultValue = "fantasy") String category, 
                         Model model){
    model.addAttribute("category", category);
    return "books.jsp";
}</pre>

Let&#8217;s then perform our test in which we don&#8217;t pass _category_ request parameter, expected value it&#8217;s gonna be &#8220;fantasy&#8221;.

<pre class="lang:default decode:true ">@Test
public void whenParameterNotSent_thenDefault() throws Exception {
    this.mockMvc.perform(get("/books"))
            .andExpect(status().isOk())
            .andExpect(model().attribute("category", is("fantasy")));
}</pre>

When request parameter is sent, the value should be, of course, the one sent and not our default.

<pre class="lang:default decode:true">@Test
public void whenParameterSent_thenParameter() throws Exception {
    this.mockMvc.perform(get("/books")
            .param("category", "java")
    )
            .andExpect(status().isOk())
            .andExpect(model().attribute("category", is("java")));
}</pre>

### 6. @RequestParam with List or array

We can also perform a call in which we specify several values for a parameter, like:

<pre class="lang:default decode:true">/books?authors=martin&authors=tolkien
</pre>

In this case, we can bind it using a _`List`_ or an array_ &#8211; _this is with a _`List`_:

<pre class="lang:default decode:true">@RequestMapping("/books")
public String books(@RequestParam List&lt;String&gt; authors,
                         Model model){
    model.addAttribute("authors", authors);
    return "books.jsp";
}</pre>

<pre class="lang:default decode:true">@Test
public void whenMultipleParameters_thenList() throws Exception {
    this.mockMvc.perform(get("/books")
            .param("authors", "martin")
            .param("authors", "tolkien")
    )
            .andExpect(status().isOk())
            .andExpect(model().attribute("authors", contains("martin","tolkien")));
}</pre>

and with an array:

<pre class="lang:default decode:true">@RequestMapping("/books")
public String books(@RequestParam String[] authors,
                         Model model){
    model.addAttribute("authors", authors);
    return "books.jsp";
}</pre>

<pre class="lang:default decode:true">@Test
public void whenMultipleParameters_thenArray() throws Exception {
    this.mockMvc.perform(get("/books")
            .param("authors", "martin")
            .param("authors", "tolkien")
    )
            .andExpect(status().isOk())
            .andExpect(model().attribute("authors", arrayContaining("martin","tolkien")));
}</pre>

### 7. @RequestParam with Map

It&#8217;s also possible to bind all request parameters in a _`Map`_ just by adding a _`Map`_ object after the annotation:

<pre class="lang:default decode:true">@RequestMapping("/books")
public String books(@RequestParam Map&lt;String, String&gt; requestParams,
                         Model model){
    model.addAttribute("category", requestParams.get("category"));
    model.addAttribute("author", requestParams.get("author"));
    return "books.jsp";
}</pre>

In this example we&#8217;ve added a _`Map<String,String>`_, the key is the request parameter name, so we just get the parameters from the _`Map`_ using their name. I think it&#8217;s worth highlight that with this approach you can&#8217;t specify which parameters are mandatory and which not and that the code can be insidious to read since might need to read it all in order to figure out which are the possible parameters used by this method.

In order to validate the snippet let&#8217;s perform a little test:

<pre class="lang:default decode:true">/books?category=fantasy&author=Tolkien</pre>

<pre class="lang:default decode:true">@Test
public void getParameter() throws Exception {
    this.mockMvc.perform(get("/books")
            .param("category", "fantasy")
            .param("author", "Tolkien")
    )
            .andExpect(status().isOk())
            .andExpect(model().attribute("category", is("fantasy")))
            .andExpect(model().attribute("author", is("Tolkien")));
}</pre>

Note that if you have multiple values for the same parameter name, you should use _[`MultiValueMap`](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/MultiValueMap.html). _In the example below, we use_`@RequestParam`_ with _`MultiValueMap<String,List<String>>`_ so that we can store multiple values for the same parameter in a _`List`_.

<pre class="lang:default decode:true">@RequestMapping("/books")
public String books(@RequestParam MultiValueMap&lt;String, List&lt;String&gt;&gt; requestParams,
                         Model model){
    model.addAttribute("category", requestParams.get("category"));
    model.addAttribute("authors", requestParams.get("authors"));
    return "books.jsp";
}</pre>

Let&#8217;s test it passing two authors and one category:

<pre class="lang:default decode:true ">/books?authors=martin&authors=tolkien&category=top250</pre>

<pre class="lang:default decode:true ">@Test
public void whenMultiValues_thenRetrieveFromList() throws Exception {
    this.mockMvc.perform(get("/books")
            .param("authors", "martin")
            .param("authors", "tolkien")
            .param("category", "top250")
    )
            .andExpect(status().isOk())
            .andExpect(model().attribute("category", contains("top250")))
            .andExpect(model().attribute("authors", contains("martin","tolkien")));
}</pre>

## Examples without @RequestParam

Based on the list of _`HandlerMethodArgumentResolver`_ configured in your application, _`@RequestParam`_ can also be omitted. If you have a look at the code of method _getDefaultArgumentResolvers()_ of _`RequestMappingHandlerAdapter`_ there is the following piece of code at the end:

<pre class="lang:default decode:true ">// Catch-all
resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), true));
resolvers.add(new ServletModelAttributeMethodProcessor(true));</pre>

Basically, it&#8217;s added to the resolvers a _[`RequestParamMethodArgumentResolver`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/method/annotation/RequestParamMethodArgumentResolver.html) _with _useDefaultResolution _set to _true._ Looking at the documentation we can see that this means that method argument that is a simple type, as defined in _[`BeanUtils.isSimpleProperty(java.lang.Class<?>)`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-),_ is treated as a request parameter **even if it isn&#8217;t annotated**. The request parameter name is derived from the method parameter name.

Let&#8217;s see in the next example what it means.

### 8. Binding without annotation

We define our controller without adding the usual _`@RequestParam`_ annotation before the method argument _`String category.<code> `</code>_

<pre class="lang:default decode:true ">@RequestMapping("/books")
public String helloWorld(String category, Model model){
    model.addAttribute("category", category);
    return "books.jsp";
}</pre>

<p class="title" title="Class RequestParamMethodArgumentResolver">
  If we run our <a href="#default-test">usual test</a> we will see that it pass!
</p>

<p class="title" title="Class RequestParamMethodArgumentResolver">
  This happens because both variable and request parameter name is &#8216;category&#8217; and in the list of default handlers, we have <em><code>RequestParamMethodArgumentResolver</code> </em>with<em> useDefaultResolution </em>set to<em> true. </em>
</p>

<p class="title" title="Class RequestParamMethodArgumentResolver">
  <em>Despite it might look convenient, I prefer to always specify the annotation since it clearly states that the argument represents a request parameter. </em>
</p>

### 9. Bind with a field of an object

In a similar way, if we pass as argument a simple pojo object that has a field &#8211; with getter and setter &#8211; named like the request parameter, the value of the request parameter will be stored in this field.

Let&#8217;s define our pojo class:

<pre class="lang:default decode:true">public class SearchFilterForm{
    private String category;

    public String getCategory() {
        return category;
    }

    public void setCategory(String category) {
        this.category = category;
    }
}</pre>

then our Controller:

<pre class="lang:default decode:true">@RequestMapping("/books")
public String helloWorld(SearchFilterForm myForm, Model model){
    model.addAttribute("category", myForm.getCategory());
    return "books.jsp";
}</pre>

and we run our [usual](#default-test) test, we will see that also in this case value of request parameter is correctly stored into _`myForm.getCategory()`_.

## pom.xml {#pom-xml}

For these tests we&#8217;ve quickly setup a Spring Book project, you can find below the pom.xml.

<pre class="lang:xhtml decode:true">&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"&gt;
	&lt;modelVersion&gt;4.0.0&lt;/modelVersion&gt;

	&lt;groupId&gt;net.reversecoding&lt;/groupId&gt;
	&lt;artifactId&gt;spring-mvc&lt;/artifactId&gt;
	&lt;version&gt;0.0.1-SNAPSHOT&lt;/version&gt;
	&lt;packaging&gt;jar&lt;/packaging&gt;

	&lt;name&gt;spring-mvc&lt;/name&gt;
	&lt;description&gt;Demo project for Spring Boot&lt;/description&gt;

	&lt;parent&gt;
		&lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
		&lt;artifactId&gt;spring-boot-starter-parent&lt;/artifactId&gt;
		&lt;version&gt;1.4.4.RELEASE&lt;/version&gt;
		&lt;relativePath/&gt; &lt;!-- lookup parent from repository --&gt;
	&lt;/parent&gt;

	&lt;properties&gt;
		&lt;project.build.sourceEncoding&gt;UTF-8&lt;/project.build.sourceEncoding&gt;
		&lt;project.reporting.outputEncoding&gt;UTF-8&lt;/project.reporting.outputEncoding&gt;
		&lt;java.version&gt;1.8&lt;/java.version&gt;
	&lt;/properties&gt;

	&lt;dependencies&gt;
		&lt;dependency&gt;
			&lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
			&lt;artifactId&gt;spring-boot-starter-web&lt;/artifactId&gt;
		&lt;/dependency&gt;

		&lt;dependency&gt;
			&lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
			&lt;artifactId&gt;spring-boot-starter-test&lt;/artifactId&gt;
			&lt;scope&gt;test&lt;/scope&gt;
		&lt;/dependency&gt;
	&lt;/dependencies&gt;

	&lt;build&gt;
		&lt;plugins&gt;
			&lt;plugin&gt;
				&lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
				&lt;artifactId&gt;spring-boot-maven-plugin&lt;/artifactId&gt;
			&lt;/plugin&gt;
		&lt;/plugins&gt;
	&lt;/build&gt;


&lt;/project&gt;</pre>

&nbsp;