---
layout: post
title: Groovy Oddities
author: Karthik Abram
tags: groovy 
---

Groovy is a popular language for the JVM with a syntax that includes and extends Java's. Groovy's appeal comes from the extended syntax - optional semicolons, type-inference (strictly speaking, Groovy is typeless), literals for lists and maps, closures, operator overloading, etc. However, these extra features are not always intuitive in their interaction, leading to some syntactic oddities. Here is a list of oddities that I've encountered. This is not an exhaustive list. If you have run into other issues, please do leave a commment below.   

### Null safety in every step

The null-safe operator allows the in-lining of nullability checks. So instead of:

```
if (student.name != null) {
	lower = student.name.toLowercase()
}
```

You can write

```
lower = student.name?.toLowercase()
```

The variable `lower` will be null if `name` is null. Armed with this powerful operator, you set about to show your newly acquired skill. You are writing a function that accepts Account objects. You know that sometimes the account may be null (if a spurious username is provided). However, if an account is non-null, you know that it has a non-null createdOn field. Your task is to get the month from this field. You write something like this:

```
void accumulateMonths(Account account) {
   def month = account?.createdOn.month
   if (month) {
	  // Do something with month
   }
}
```  

You run the program and upon receiving a `null` Account object, you see an exception!

```
Exception in thread "main" java.lang.NullPointerException: Cannot get property 'month' on null object
at org.codehaus.groovy.runtime.NullObject.getProperty(NullObject.java:57)
``` 

Why did Groovy not stop when it evaluated `account?.` ? Why is it complaining about not being able to get the month property? The exception was thrown because we incorrectly interpreted the semantics of the `?.` operator. The null-safe operator in fact does not abort the current chain of calls on encountering a null. It is better translated to the construct below.

```
def a = account == null ? null : account.createdOn
def month = a.month
if (month) {
}
``` 

The null-safe operator is a close-cousin of the ternary operator, not the `if` condition!

### Applying null-safe retrieval to map entries.

In Groovy, you can access map entries as `map[key]` and list entries as `list[index]`. So if we had a `Map<String, List<Integer>>`, and we want to make sure a map entry is not null before we access the list, should we be able to use the null-safe operator? 

```
class NullSafeWithMap {

    static void main(def args) {     
        Map<String, List<Integer>> myMap = ['a': [1, 2, 3], 'b': null, 'c': [4, 5, 6]]
        println (myMap['b']?[0]) // Syntax here here.
    }
}
```
Unfortunately you will get a syntax error since Groovy thinks the `?` is part of the ternary operator. This is because the null-safe operator is `?.` not just a `?` Putting a period after the question-mark will not help either.

```
myMap['b']?.[0] // Still a syntax error.
``` 


If you replace the list subscript operator with the `get()` method, the code will work:

```
println (myMap['b']?.get(0))
```

### An equation to get NPE

Here is a brain teaser. What is the output of the program below?

```
static void main(def args) {
     println 4 + 5 * 3
}

```
Alright, that's not much of a brain-teaser. The answer is obviously 19. Turns out, what I wanted was really (4 + 5) * 3 so that I get 27. That's easy:

```
static void main(def args) {
	println (4 + 5) * 3
}
```

Think the answer will be 27? Think again! Lost in the focus on the math equation is the fact that you were able to write `println 4 + 5 * 3` in the first place because Groovy lets you drop parenthesis if you have a single argument and the call is unambiguous. So instead of printing out the product of 4 + 5 and 3, Groovy is attempting to multiply the result of `println (4 + 5)` and 3. `println` has a return type of void which when forced into a return value becomes null. So at runtime, Groovy throws an exception stating that it cannot call `multiply()` on a null object.  

### Operator precedence rules

Groovy supports operator overloading and provides several pre-defined operators to make your code succinct, such as the `<<` operator to add to lists.

```
List fruits = []
fruits << 'Apples' // fruits = ['Apples']
```

While writing your code, you come across a case where if citrus is true, you want to add 'Oranges' but otherwise you want to add 'Apples'. Eager to show off your newly acquired Groovy skill, you write the following:

```
List fruits = []
boolean citrus = ... 
fruits << citrus ? 'Oranges' : 'Applies'

println fruits

```

As you extol the virtues of Groovy to your colleagues, you look at the output and are horrified to find:
`[true]`

What just happend? Well, turns out the operator precedence puts the left-shift operator at a higher precedence than the ternary operator. So the expression you really wrote is:

```
(fruits << citrus) ? 'Oranges' : 'Apples'
```

In Groovy, a list has a `true` value if it is non-null and non-empty. So 'Oranges' was selected as the value of the expression and sent to ... /dev/null? To fix the issue, you have to write 

```
fruits << (citrus ? 'Oranges' : 'Apples')
```


### Block scoped local variables

Seasoned programmers develop habits to prevent certain types of errors. One of those habits is to use blocks to scope local variables so that copy-paste of the code (hopefully confined to some hastily written unit test) doesn't result in redefinition of the variables, which invariably leads to an attempt to rename the variables, which occasionally results in one rename being left off, which results in a hard-to-find bug. So we write

```
{
   int a = 5;
   callSomeMethod(a);
}
{
   int a = 10;
   callSomeMethod(a);
}
```
There are more legitimate uses for the block-scope in methods, such as to ensure that logic that follows does not rely on some previous variable. When translated to Groovy, Java code with blocks fail because Groovy thinks that you have defined closures and therefore skips over them. The workaround?

```
1.times {
   int a = 5;
   callSomeMethod(a);
}
1.times {
   int a = 10;
   callSomeMethod(a);
}
```     
The above construct is arguably a hack, but it works nevertheless.

### Map literal oddities

Groovy supports map literals. This allows you to create a map with a compact and readable syntax.

```
Map<String, Integer> myMap = ['a': 25]
myMap['b'] = 35
int d = myMap['a']
```

Armed with this new-found knowledge, you proceed to construct a map as follows.

```
String name = 'joe'
Map<String, Integer> myMap = [name: 25]
...
int value = myMap[name]
```

You run your program to discover a runtime exception saying that null cannot be converted to `int`! Upon further inspection with a debugger, you discover that your map is `['name': 25]` not `['joe': 25]`. What happened? Turns out you can't have variables in a map initialization. To achieve the intent above, you have to use a GString variable or enclose the variable in parenthesis.

```
String name = 'joe'
Map<String, Integer> myMap = ["$name": 25] // or [(name): 25]
```
Map initialization cannot take the return values of functions or enums either. These also need to be turned into a GString expression or wrapped in parenthesis.

### Primitive arrays

If you argue that Java's primitive arrays are an aberration, you win. Now lets deal with with the real world problem presented by the fact that many java api have been written to accept primitive arrays. Since Groovy uses `[]` to represent lists, how do you define a primitive array instead? 

Given the java method

```
public class SomeClass {

    public static void soStuff(int[] values) {
        for (int i : values) {
            System.out.println(i);
        }
    }
}
```

the following Groovy code will fail.

```
def values = [1, 2, 3]
SomeClass.soStuff(values) 
```

To coerce `values` into a primitive array, use the `as` operator with `<type>[]` type definition.

```
def values = [1, 2, 3]
SomeClass.soStuff(values as int[])
```

### Type-lessness

Groovy supports duck typing. Yes, but what kind of duck allows this?

```
static int abc = new AtomicInteger()
```
Remember that in Groovy, type declarations in your code are completely ignored. So the compiler will happily compile the above. Groovy 2.0 introduced specific annotations, `@TypeChecked` and `@CompileStatic` to alleviate some of these bugs. There are some caveats to be aware of when using these annotations. Refer to this [presentation](http://portal.sliderocket.com/vmware/Groovy-2-0-Static-Type-Checking-and-Compilation) for a good overview.  


### BigDecimal everywhere

Sometimes features built with good intent can have unintended consequences. The next three topics present examples of such features in Groovy. Groovy converts all division to BigDecimal. This is especially useful in financial services apps. However, it converts all division to BigDecimal as I found out. I was writing code to determine the number of threads to use to process a list of objects. I wanted the thread count to be one-tenth the list size but with a minimum of 1 and a maximum of 50. 

```
int threads = Math.max(1, Math.min(list.size() / 10, 50))
```

When I ran the program, I got an exception stating, `Cannot resolve which method to invoke for [class java.math.BigDecimal, class java.lang.Integer] due to overlapping prototypes between ...`

Uh? I was expecting integer division and Groovy instead offered a BigDecimal and couldn't find an appropriate `Math.min()` method. The code needs to be rewritten to

```
int threads = Math.max(1, Math.min((list.size() / 10).intValue(), 50))
```

### Groovy Truth

Groovy evaluates truth as many things - null objects are false, empty lists are false, zero is false. Takes you back to the days of C/C++. People bemoaned the unexpected evaluations of clever expressions in C/C++. Java came along and eschewed all cleverness. People bemoaned the verbose conditional expressions in Java. Groovy came back with C/C++ style truth evaluation. 

In a payment app, we want to make sure the amount has been defined. So we write a test of nullability.

```
if (!amount) {
	throw new IllegalArgumentException(...
}
```

Except somehow, the amount ends up being exactly zero (lets say this is the amount to be charged and a coupon made the amount zero). Well, the code unexpectedly fails.


### Safe-navigation operator

Groovy allows you to test for nullability and continue an operation by using the safe-navigation operator `?`. This is an operator that is frequently abused (its use masquerading for thoughtful analysis of nullability) and sometimes results in side-effects too.

Consider the following condition

```
if (!someObject.someProperty?.taxable) {
}
``` 

The author's intent was that if taxable is false, some piece of code should be executed. However, if someProperty itself is null, in this particular case, the code should not have been executed. The author realized that someProperty is a nullable field and therefore came up with the above construct. If someProperty is null, the expression evaluates to true! The actual condition required was

```
if (someObject.someProperty && someObject.someProperty.taxable) {
}
```

### Language philosophy

Groovy introduced a number of new language constructs to the JVM and offered a terse and powerful language to program in. Groovy is particularly suited to unit-tests and DSLs and frameworks such as Grails. What is not apparent in the power of Groovy is a trade-off that has been made between discipline and power. Programmers can abuse the power of Groovy with greater ease than Java and the result is bad code that is tersely defined versus bad code that is verbosely defined. The ease of use of maps and lists in Groovy leads to their excessive use often at the expense of proper Object-Orientedness.

Its been an interesting journey since the days of C, C++. C and C++ allowed all kinds of clever programming tricks and shortcuts. Developers grew tired of the side-effects of the tricks and unfortunately, at least with Java, the language designers pivoted to the other extreme. Groovy in some respects represents a swing back to the C/C++ paradigm. Language designers are now attempting to strike the right balance between concise expression vs. correctness, static type vs. verbosity and terseness vs. comprehensibility. Some of the newer languages like [Dart](https://www.dartlang.org/), [Swift](https://developer.apple.com/swift/) and [Ceylon](http://ceylon-lang.org/) exemplify such a design trend. Ceylon in particular is an exciting JVM language that is based on the philosophy that code is read more often than written. Ceylon has one of the most [exciting type-systems](http://blog.jooq.org/2013/12/03/top-10-ceylon-language-features-i-wish-we-had-in-java/) and I for one am eagerly looking forward to using it.    
 