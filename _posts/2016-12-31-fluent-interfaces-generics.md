---
layout: post
title: Fluent Interfaces without Recursive Generics
author: Karthik Abram
tags: java 
---

Fluent interfaces are an API design style that allow you to be expressive while being terse. Consider the following *pseudo* example API to split a string.

{% highlight java %}

Splitter splitter = new Splitter(",");
splitter.setOmitEmptyStrings(true);
splitter.setTrimOutput(true);
List<String> parts = splitter.splitToList(input);

{% endhighlight %}

That is four lines packed with a lot of noise. The two setters could have been written as parameter-less methods but I wanted to stick to Javabean specifications here. Now compare this with the fluent version of [Splitter](https://github.com/google/guava/wiki/StringsExplained#splitter) from [Google Guava](https://github.com/google/guava).

{% highlight java %}

List<String> parts = Splitter.on(",") //
	.omitEmptyStrings() //
	.trimResults() //
	.splitToList(input);

{% endhighlight %}

Still four lines but now each line clearly conveys intent in a very "fluent" dialogue like manner. So armed with an understanding of the elegancy that fluent interfaces bring to the table, you set out to create a fluent version of your new animal API. You have written up the following classes with fluent API:

{% highlight java %}

public class Animal {

    public Animal eat(Food food) {
        food.consumedBy(this);
        return this;
    }
    
    
    public Animal run(int distance) {
        // some logic here
        return this;
    }
}

public class Dog extends Animal {
    
    public Dog bark() {
        // logic
        return this;
    }
    
    
    public Dog chaseTail() {
        // logic here
        return this;
    }
}

{% endhighlight %}

Now you attempt to use your newly created fluent API and write up the following client code:

{% highlight java %}

public void someMethod() {
	Animal a = new Dog().eat(bone).bark(); // Cannot resolve method bark ???
}

{% endhighlight %}

You are suddenly perplexed. Why can't you call method bark()? It is because eat returns an instance of Animal, not Dog and so even though the runtime instance created is a Dog, the static type checking only knows that you have an instance of Animal. This particular situation is easily resolved by switching the two methods:

{% highlight java %}

public void someMethod() {
	Animal a = new Dog().bark().eat(bone);
}

{% endhighlight %}

In a real world API, we may not have the luxury of switching methods like this (what if the dog really only likes to bark after eating!) and requiring particular ordering of calls makes for an ugly API. The standard solution in such a case is to introduce generics with a particular twist - recursive generics.


{% highlight java %}

public class Animal<A extends Animal<A>> {

    public A eat(int food) {
        //food.consumedBy(this);
        return (A)this;
    }


    public A run(int distance) {
        // some logic here
        return (A)this;
    }
}

public class Dog extends Animal<Dog> {

    public Dog bark() {
        // logic
        return this;
    }


    public Dog chaseTail() {
        // logic here
        return this;
    }
}

// in some method ...
Animal<Dog> dog = new Dog().eat(3).bark();

{% endhighlight %}

Problem solved? Well, almost. Until you realize that if you know include Animal as a member of any class, you bring along baggage - in the form of [recursive generics](http://stackoverflow.com/questions/26304527/recursive-generic-and-fluent-interface) that you have to apply to the container class (or method) and this can easily mushroom into very complex generic definitions. There is an alternate way of building fluent interfaces with inheritance but without generics by accepting a slight compromise. We do this by introduce a very specific typing method that allows us to switch the static type.

{% highlight java %}

public class Animal {

    public Animal eat(int food) {
        //food.consumedBy(this);
        return this;
    }


    public Animal run(int distance) {
        // some logic here
        return this;
    }


    // A way to cast our return type into a specific sub-type.
    public <T extends Animal> T typed() {
        return (T)this;
    }
}

public class Dog {
	...
}

// caller
Animal dog = new Dog().eat(3).<Dog>typed().bark();

{% endhighlight %}

While not perfect, this technique allows us to switch between methods of the parent or child class and not incur the added design-time overhead of generics.

