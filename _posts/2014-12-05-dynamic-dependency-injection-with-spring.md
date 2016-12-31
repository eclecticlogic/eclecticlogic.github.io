---
layout: post
title: Dynamic dependency injection with Spring
author: Karthik Abram
tags: java spring
---

Out of the box, Spring provides various techniques to define your beans and their dependencies - XML files, Java annotations and since version 4.0, even using [groovy scripts](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/support/GenericGroovyApplicationContext.html). Furthermore, you can control the creation of the beans using [FactoryBean](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/FactoryBean.html) and bean [post-processors](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html). Still, there are times when it would be very convenient to create an instance of some random class and have its dependencies satisfied at runtime without Spring knowing anything about that class a priori. 

If you are willing to accept a few limitations - only bean dependencies, not simple scalar properties, and only those annotated with `@javax.inject.Inject`, then the following code creates a dependency injector that you can use to dynamically "bind" random objects.

{% gist eclecticlogic/f384dcfd967a8537a890 DependencyInjector.java %}

The dependency injector itself should be a spring bean and available to your code that wants to inject something else. In addition, you need to have an instance of [AutowiredAnnotationBeanPostProcessor](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.html). This bean is automatically included if you have the annotation-config directive in your Spring context file:

{% highlight xml %}

	<context:annotation-config />

{% endhighlight %}

You can now use the dependency injector to bind dependencies at runtime as follows:


{% highlight java %}

	public class MyObject {
		@Inject SomeOtherObject other; // SomeOtherObject is spring managed
        ...
    }


	...
    MyObject withDependencies = dependencyInjector.bind(new MyObject());

{% endhighlight %} 

