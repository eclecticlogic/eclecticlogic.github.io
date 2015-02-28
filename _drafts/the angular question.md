---
layout: post
title: The Angular Question - The constant winds of change
author: Karthik Abram
tags: architecture
---

Architect's long for the opportunity to work on a greenfield project. Free from the shackles of all things legacy, we imagine using cutting-edge technologies, languages and frameworks, not to mention the expert guidance of Mr. Propeller head, to envision and fashion the quintessential system. Such idealism and accompanying euphoria lasts at least for a fleeting moment.

The reality of making actual technology stack choices is arduous and often contentious. While some choices tend to be dictated by considerations such as in-house expertise and familiarity (operating system, platform, language, database), others, especially of late, the choice of framework for the web interface is fraught with personal preferences, prejudice, hype and uncertainty. JavaScript based frameworks are currently the rage. Indeed, even the mention of a non-JavaScript based framework in some quarters is considered beyond faux-pas. In the last couple of years we have had a number of frameworks claim the spotlight and just as quickly vanish, sidelined as relics by progress and advancement. ExtJS, KendoUI, BackboneJS, EmberJS and the current reigning king - AngularJS. It does not take a prophet or the finger of God writing on King Belshazzar's wall, for one to predict Angular's inevitable relegation. Vying to replace Angular are Polymer, ReactJS and untold many incubating in Github. 

The question to be asked then is this. How does one make a responsible choice of framework in the midst of such  rapid progress and uncertainty? It is worth repeating the obvious - no single framework will remain technologically viable and contemporarily desirable for any appreciable length of time. Let me also state the other obvious fact that our ego prevents us from readily admitting - we do not have the vision or the clairvoyance to pick the framework that will stay relevant the longest. I do believe we can use a few heuristics to guide us - a framework that is a one-man show is likely to not have a long shelf time; large companies, Google, Apple, Microsoft and Facebook are more likely support their frameworks for a long time (with notable counter-examples). So how then do we make a decision?

There are no simple answers to this question, but we can formulate common-sense guidelines and principles. Begin by asking yourself the following questions:

1) What does the business requirement really dictate? Do we really need to build a single-page application or would a more traditional server-side MVC style architecture work?
2) How large a system are we dealing with? 
3) What is the skill set of the team?
4) How well does the framework lend itself to supporting good software design principles? 

Of the questions above, number 4 is the most important one. This is because a well-design, modular, layered system with clean separation of concerns can weather the winds of change better than a poorly designed one. And frameworks directly and indirectly contribute to our ability to write well engineered systems. Case in point: AngularJS. I believe the AngularJS team's decision to adopt such a radical redesign for 2.0 is a telling proclamation of the design failures of the 1.0 code base. There are quite a few well written technical blogs on the shortcomings of AngularJS. I'm going to highlight just one (from a [blog](http://www.binpress.com/blog/2014/06/26/polymer-vs-angular/) comparing Angular and Polymer).

A custom element in Polymer is defined as follows:
```
<polymer-element name="user-gravatar" attributes="email">
  <template>
    <img src="https://secure.gravatar.com/avatar/{{gid}}" />
  </template>
  <script>
    Polymer('user-gravatar', {
      ready: function() {
        this.gid = md5(this.email);
      }
    });
  </script>
</polymer>
```
Without any introduction to Polymer, the words "custom" and "element" trigger certain expectations in a programmer's mind:

1. On encountering the element `<polymer-element>` with a `name` attribute, one gets the indication that this is the definition of the custom element and it is likely `<user-gravatar>`.
2. The `<template>` element conveys the body of this custom element.
3. The script perhaps provides the behavior. `ready` as a function could be indicative of an initializer.


Compare the above to the Angular definition:

```
app.directive('user-gravatar', ['md5', function() {
  return {
    restrict: 'E',
    link: function(scope, element, attrs) {
      scope.gid = md5(attrs.email);
    },
    template: '<img src="https://gravatar.com/avatar/{{gid}}" />'
  };
}]);
```

The terms "directive", "restrict" and "link" don't seem to have any semantic correspondence with the idea of a custom-element (I'm not going to comment on the string 'md5'). What does `restrict: 'E'` convey to the Angular ignorant? The question of JavaScript framework selection, however, is larger than the particular framework and design choice that Angular represents. Jeff Atwood wrote an insightful article in 2009 titled [All Programming is Web Programming](http://blog.codinghorror.com/all-programming-is-web-programming/). In the article he quotes Michael Braude lamenting that the popularity of web programming is leading to mediocrity. Atwood states that even if Braude's lament is true, it is irrelevant because everything is moving to the web. Then he coined the now famous idiom: "Any application that can be written in JavaScript, will be eventually written in JavaScript."

I would like to propose a corollary to the above:

Most applications written in JavaScript, will eventually be difficult, if not impossible to maintain.

I say this because JavaScript at present is a language ill-suited to the development of large, complicated systems. This is a bold, but rational, assertion as evidenced by the fact that Microsoft announced TypeScript after embracing JavaScript, Google produced Dart and now with Angular 2.0, AtScript and even Facebook, with ReactJS, is eagerly looking forward to supporting EC6. JavaScript lacks type-safety (a debate for another day; again, AtScript, TypeScript, Dart all make a strong statement in this regard), modularity (beyond the prototype hack substituting for a class), code organization (packages and interoperable third-party libraries), dependency management and tooling (in-line docs, standardized debuggers, etc). All of the above are in an active state of evolution which in itself adds to the entropy of the JavaScript eco-system.

**REVIEW FROM HERE**

Faced with such a constant change, a greenfield project being undertaken can make the following choices:

1. Go with a language that compiles to JavaScript. In other words, treat JavaScript as the [assembly language](http://www.hanselman.com/blog/JavaScriptIsWebAssemblyLanguageAndThatsOK.aspx) of the web. TypeScript, CoffeeScript, Clojure, Dart, etc. all seem to be designed for such a conversion (with Dart it is a secondary goal). Even newer languages such as Ceylon and Kotlin are explicitly targeting JavaScript cross-compilation as a design goal. If you go this route, you have a number of options. You can use a JavaScrpt "like" language such as TypeScript or CoffeeScript and build using JS frameworks that are polygot or polygot supportive. If you are feeling more adventurous (and are confident of Google's plans; history is not your friend in this regard), you can go with Dart (or Dart with Polymer or AngularDart). Lastly for traditional Java shops, GWT remains a viable option (first announced in May of 2006, it is the Methuselah of web frameworks). It is very encouraging to see a still-strong community around GWT and a number of commercial enterprises involved in its steering. GWT 3.0, expected in Q4 of this year, has some grand ambitions which Java developers are going to find hard to ignore.
2. Go with a pure-JavaScript approach. The established contenders like KendoUI, Ember, Angular will need to be evaluated. Angular has most likely Osborned itself with the 2.0 announcement. I know of shops that are launching AngularJS 1.x based systems that are facing the reality that have a legacy system on day one and are starring at the prospect of a rewrite in the near future. I for one find it hard to recommend AngularJS 1.x. If you have the luxury of time, wait for 2.0 expected later this year.
3. Go with a "minimal" framework approach. If you are wondering why I didn't include ReactJS in point (2), its because I believe ReactJS with its component-based approach and a deliberate decision to do a few things, but do them well deserves recognition as a different choice. I have been a proponent of component based frameworks for a long time (I've used ASP.NET, Tapestry and GWT based components with great success). Component based frameworks reduce the cognitive burden on developers by allowing abstractions to naturally be built and re-used. 

