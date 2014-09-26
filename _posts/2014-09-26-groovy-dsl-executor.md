---
layout: post
title: Groovy DSL Executor
tags: java groovy DSL
author: Karthik Abram
---

Modern JVM languages all support writing DSLs to allow for more abstraction representation and modeling of the problem domain. [Scala](http://www.scala-lang.org/old/node/1403) has them, [Groovy](http://groovy.codehaus.org/Writing+Domain-Specific+Languages) has them and the up-and-coming [Ceylon](http://ceylon-lang.org/documentation/1.0/tour/named-arguments/) has them too. I'm personally very excited about Ceylon and its rich type system and am looking forward to trying it out in the near future.

In the meantime, I wanted to build a Groovy DSL for a project to facilitate some elaborate data setup. The GroovyBuilder mechanism wasn't to my liking as it seemed to closely tied to object instantiation and the canonical syntax for construction. While researching options I came across [Steven Devijver's](http://groovy.dzone.com/news/groovy-dsl-scratch-2-hours) post on creating Groovy DSLs by supplying a closure that bootstraps the script execution using delegate modification and prioritization. The idea while intriguing had two aspects that didn't fit well with my requirement:

1. There wasn't an easy way to supply the closure to a self-contained "DSL executor."
2. I'd have to provide closure definitions for every top-level method I wanted to have in my DSL.


My ideal solution would involve giving a `DSLExecutor` object a script reference and an instantiated handler object that had the method definitions for the script. Passing an instantiated handler would allow me to provide a managed bean with dependencies satisfied. To achieve this goal I decided to go digging through Spring's `GenericGroovyApplicationContext` that provides a DSL for Spring context configuration and the `groovy.lang.Script` class itself. Spring groovy application context takes advantage of the fact that the `groovy.lang.Script` class has an overridden invoke method that attempts to see if the `Binding` object supplied as a closure definition of the same name as the method that just failed to execute:

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

Trying to combine ideas from Steve's post, Spring and groovy.lang.Script implementations, I realized that if I supply the Script's metaclass with a [methodMissing](http://groovy.codehaus.org/Using+methodMissing+and+propertyMissing) closure, I can effectively route unmatched methods to any arbitrary object. The result is a simple standalone DSL executor:

{% gist eclecticlogic/2b8a8d78dab281a26378 DSLExecutor.groovy %}

The `DSLExecutor.execute()` method accepts a filename that is resolved using Spring's resource loader (you can therefore prefix the filename with `classpath:` or other prefixes that Spring supports) and a handler object that is expected to have methods for your constructs in the DSL. The `execute()` method returns a map of variables returned by the script.

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

We can further extend the handler paradigm by making it hierarchical, wherein the handler delegates processing of some methods to a different handler. For example, if our DSL had a section dealing with users, we may want to be able to write a construct such as:

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

We may want to segregate the responsibility of all user related methods to a UserHandler class. This is easily achieved within the primary handler by setting the delegate on the closure supplied to the users method.

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

The `DSLExecutor` further simplifies writing custom DSLs. Happy DSL-ing? 