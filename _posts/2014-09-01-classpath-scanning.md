---
layout: post
title: Classpath scanning with Spring
author: Karthik Abram
tags: java spring
---

Spring has a number of utilities that the framework ships with. One of them is a classpath scanner provided by `ClassPathScanningCandidateComponentProvider`. That is a long name and a mouthful. The class allows you to enable auto-discovery features in your own framework or code. Here is a simple example to demonstrate its use:

{% gist eclecticlogic/7551622 SpringClasspathScanner.java %}

The constructor takes a boolean parameter to apply default filters or not. Default filters are based on annotations for `@Component`, `@Repository`, `@Service` and `@Controller`. Once you have your provider, you give it filters to zero-in on the types you are interested in. Filters implement the `TypeFilter` interface and Spring helpfully ships with a number of useful ones:

- AnnotationTypeFilter: filters based on annotations on the class.
- AssignableTypeFilter: filters that match based on superclass or interface.
- RegexPatternTypeFilter: matches a fully qualified class name against a regular expression.

Spring also has an AbstractClassTestingTypeFilter that allows you to write your own matcher against class metadata that spring provides for each Class/Interface that it finds.

Once you have your filters defined, you simply give the scanning provider a root package to search within and it returns you a list of BeanDefinition objects for matches. You may incorrectly assume, because of the name BeanDefinition, that this is restricted to beans in a Spring context. Thankfully, it is not, as that would severely restrict the utility value of this class. The BeanDefinition is simply a container of useful information about the class including the `Class` instance.
  
	 