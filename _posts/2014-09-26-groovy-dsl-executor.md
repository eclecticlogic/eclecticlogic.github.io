---
layout: post
title: Groovier Groovy DSL
tags: java groovy DSL
author: Karthik Abram
---

DSLs allow a programmer to tailor-make keywords and syntax to express a higher abstract representation of a problem domain. Recognizing the power of DSLs, most modern JVM languages have taken to support them. [Scala](http://www.scala-lang.org/old/node/1403), [Groovy](http://groovy.codehaus.org/Writing+Domain-Specific+Languages) and the up-and-coming [Ceylon](http://ceylon-lang.org/documentation/1.0/tour/named-arguments/) all support writing DSLs.

Recently I needed to build a Groovy DSL to facilitate some complex data setup for executing automated scenario tests. The GroovyBuilder mechanism wasn't to my liking as it seemed too closely tied to object instantiation and the canonical constructor syntax. While researching other options I came across [Steven Devijver's](http://groovy.dzone.com/news/groovy-dsl-scratch-2-hours) post on creating Groovy DSLs. Steve's technique works by defining a closure to bootstrap the script execution and redirecting method resolution using delegate modification and prioritization. The idea while intriguing had two aspects that didn't fit well with my requirement:

1. There wasn't an easy way to supply the closure to a self-contained "DSL executor."
2. I would have to provide closure definitions for every top-level method I wanted to have in my DSL.

My ideal solution would involve giving a self-contained `DSLExecutor` object a script filename and a handler object that had the method definitions for the script. I decided to dig through Spring's `GenericGroovyApplicationContext` that provides a DSL for Spring context configuration and the `groovy.lang.Script` class for inspiration and clues to achieve this. Spring's Groovy context loader takes advantage of the fact that the `groovy.lang.Script` class, using an overridden invoke method, attempts to see if the `Binding` object has a closure definition of the same name as the method that just failed to resolve:

{% highlight groovy linenos %}

public Object invokeMethod(String name, Object args) {
    try {
        return super.invokeMethod(name, args);
    }
    // if the method was not found in the current scope (the script's methods)
    // let's try to see if there's a method closure with the same name in the binding
    catch (MissingMethodException mme) {
        try {
            if (name.equals(mme.getMethod())) {
                Object boundClosure = getProperty(name);
                if (boundClosure != null && boundClosure instanceof Closure) {
                    return ((Closure) boundClosure).call((Object[])args);
                } else {
                    throw mme;
                }
            } else {
                throw mme;
            }
        } catch (MissingPropertyException mpe) {
            throw mme;
        }
    }
}

{% endhighlight %}

Combining ideas from Steve's post, Spring and the groovy.lang.Script implementations, I realized that by supplying the  `groovy.lang.Script`'s metaclass with a [methodMissing](http://groovy.codehaus.org/Using+methodMissing+and+propertyMissing) closure, I can effectively route unmatched methods to any arbitrary object. The result is a simple standalone DSL executor:

{% gist eclecticlogic/2b8a8d78dab281a26378 DSLExecutor.groovy %}

The `DSLExecutor.execute()` method accepts a script filename and a handler object. The script is resolved using Spring's resource loader (you can therefore prefix the filename with `classpath:` or other prefixes that Spring supports). The handler object is expected to have methods for your constructs in the DSL. The `execute()` method returns a map of variables returned by the script.

A simple example of a handler is shown below:

{% highlight groovy linenos %}

class MyHandler {

   def printMe(String name, Closure cl) {
		println "Hello $name"
        cl(name)
   }
}

{% endhighlight %}

The above code allows a simple DSL of the form:

{% highlight groovy linenos %}

printMe "Dude" {
    println it
}

{% endhighlight %}

We can extend the handler paradigm by making it hierarchical, wherein the handler delegates processing of some methods to a more specialized handler. For example, if our DSL had a section dealing with users, we may want to be able to write a construct such as:

{% highlight groovy linenos %}

printMe "Dude" {
   println it
}

users {
    tom = create "Tom"
    login tom {
        it.setRememberMe()
    }
}

{% endhighlight %}

We may want to segregate the responsibility for all user related methods to a UserHandler class. This is easily achieved within the primary handler by setting the delegate on the closure supplied to the users method.

{% highlight groovy linenos %}

class UserHandler {
   // ... various methods
}

class MyHandler {

   def printMe(String name, Closure cl) {
	println "Hello $name"
        cl(name)
   }

   def users(Closure c) {
       UserHandler handler = new UserHandler()
       c.delegate = handler
       c.resolveStrategy = Closure.DELEGATE_FIRST
       c()
   }
}

{% endhighlight %}

We thus have yet-another-DSL construction paradigm using Groovy. I hope other folks find this useful and are able to create some elegant tailor-made DSLs with it.
