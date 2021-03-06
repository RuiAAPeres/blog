---
layout: post
title: "Xembly, an Assembly for XML"
date: 2014-04-09
tags: xembly xml xsl xsd
description: An imperative language for XML data and schema manipulations
keywords:
  - xml
  - xsl
  - xembly
  - assembly for xml
  - xml modification
  - xml manipulation
  - xpath
---

I use XML in almost every one of my projects. And, despite all the fuss about JSON/YAML, I honestly believe that XML is one of the greatest languages ever invented. Also, I believe that the beauty of XML reveals itself when used in combination with related technologies.

For example, you can expose your data in XML and render it for the end-user using [XSL stylesheet](http://www.w3.org/Style/XSL/).
Another example would be when you validate the same data,
before rendering, to ensure that the structure is correct. You can do this with the [XSD](http://www.w3.org/TR/xmlschema11-1/) schema.
Alternatively, you can pick specific data elements from the entire document by using
[XPath](http://www.w3.org/TR/xpath/) queries.

Essentially, these three technologies, XSL, XSD schema and XPath, are what makes XML so powerful:

However, there can be times when XML falls short. For instance, imagine you have
an existing document that needs to be modified just slightly. For example, let's use the following:

{% highlight xml linenos=table %}
<accounts>
  [...]
  <acc id='34'>
    <name>Jeffrey</name>
    <balance>305</balance>
  </acc>
  <acc id='35'>
    <name>Walter</name>
    <balance>50090</balance>
  </acc>
  [...]
</accounts>
{% endhighlight %}

The above code represents a list of accounts. Each account has its own `id` and several
child elements. In our example, we need to find the account belonging to `Jeffrey`
and increase its balance by `500`. How would we do this?

Well, there are a few possible solutions:

 * SAX-parse the document, change the balance and save the stream;
 * DOM-parse it, find the element with XPath, change the value and then print it;
 * apply a parametrized XSL stylesheet;
 * apply XQuery small script to make changes

All of these methods have their own drawbacks. However, all of them have one particular problem in common &mdash; they are very verbose. With each of the above methods, you need at least a page of code to perform this rather simple operation. Furthermore,  if the logic of the operation becomes more complex, the amount of needed code grows much faster than you may expect.

Simply put, XML lacks a tool for primitive data manipulations within
a document. Perhaps, it is this shortcoming that makes XML unpopular with some.

Anyway, here is a tool I created a few month ago: [Xembly](http://www.xembly.org).
It is an imperative language with a few simple directives and resembles [Assembly](http://en.wikipedia.org/wiki/Assembly_language)
in style. Thus, the name - Xembly. With Xembly, there are no loops, conditions or variables - just a sequence of directives with arguments.

Let's create a simple example. Say, for instance, we want to add a new account number `36`
to our list document. The code would look like:

{% highlight asm linenos=table %}
XPATH '/accounts';
ADD 'account';
ATTR 'id', '36';
ADD 'name';
SET 'Donny';
UP;
ADD 'balance';
SET '3400';
{% endhighlight %}

The above should be intuitively clear, but I'll explain just in case. First, the `XPATH` directive points us
to the element found by the "/accounts" XPath query. This will be our root element. We assume here that
it exists in the document. Therefore, if it is absent, our Xembly script will
fail with a runtime exception.

Next, the `ADD` directive on line 2 creates a new XML element without any children
or attributes. Then, the `ATTR` directive sets an attribute for this element. The code then adds
the new child element `name` and sets its text value to `"Donny"` using the `SET` directive. Finally, we move our pointer back to `account` element using `UP`, add the `balance` child element and set its value to `"3400"`.

Our balance changing task can be expressed in Xembly with the following code:

{% highlight asm linenos=table %}
XPATH '/accounts/account[name="Jeffrey"]/balance';
XSET '. + 500';
{% endhighlight %}

The `XSET` directive sets the element text value, similar to `SET`, but
calculates it beforehand using the provided XPath expression `. + 500`.

Xembly performs all manipulations through DOM. Consequently, Xembly can be
implemented inside any language that has a built-in DOM implementation.

In the meantime, there is only one implementation of Xembly language
&mdash; in Java. Here is how it works:

{% highlight java linenos=table %}
Iterable<Directive> directives = new Directives()
  .xpath("/accounts")
  .add("account")
  .attr("id", "36")
  .add("name").set("Donny").up()
  .add("balance").set("3400");
new Xembler(directives).apply(document);
{% endhighlight %}

In this snippet, I'm using a supplementary script builder, `Directives`, which
enables generation of directives in a fluent way. Then, I use `Xembler` class,
which is similar to "assembler", to apply all specified directives
to the `document` object of class `org.w3c.dom.Document`.

Additionally, Xembly can be used to build XML documents from scratch and as a replacement for traditional DOM building. A quick example:

{% highlight java linenos=table %}
System.out.println(
  new Xembler(
    new Directives().add("html")
      .add("head")
      .add("title")
      .set("Hello, world!")
  ).xml()
);
{% endhighlight %}

The above snippet produces the following output:

{% highlight xml linenos=table %}
<html>
  <head>
    <title>Hello, world!</title>
  </head>
</html>
{% endhighlight %}

For me, this appears to be more simple and compact.

As usual, your bug reports and suggestions are always welcomed. Please send to
[Github issues](https://github.com/yegor256/xembly/issues) :)
