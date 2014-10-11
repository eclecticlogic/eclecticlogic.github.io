---
layout: post
title: Of Slurpees and Bean Burritos 
draft: true
author: Karthik Abram
tags: oo design humor
---

Groovy ships with some interestingly named utility classes - `ConfigSlurper`, `XmlSlurper` and `JsonSlurper`. I wonder if the engineer wrote this class while hanging out at a 7-Eleven drinking a slurpee!

![]({{ site.baseurl }}public/images/blog/slurpee.jpg)

The class should make a slurping noise as it runs complete with a louder noise as it ends - similar to an annoying kid sucking at the last bit of drink from a mostly empty cup filled with ice. One has to wonder about the wisdom behind calling a class Slurper in a professional software library. But perhaps such concerns are not valid when dealing with a language that is itself called Groovy.  The software world has a penchant for coming up with interesting names. Apart from culinary and beverage related names such as Java, Beans, Cocoa and Guava topped with Sugar and eliciting a Yum response, we also have reptilian and warrior names like Python, Boa, Ninja and others. Indeed a simple perusal of Github leads to encounters with Pigshell, Nightmare and Phantom. Not to be left behind, the hardware world is recalibrating from sophisticated names like Centrino and Ivy Bridge to Raspberry Pi and Beaglebone.

In object oriented design, coming up with a good name is a hard problem. In fact, most of the arguments in OO design are related to naming objects. As a professor once said, there are really only two problems in Computer Science:

1. Naming objects
2. Cache invalidation and
3. Off by one errors.

Having had my share of endless arguments over class names, I decided to mine open source software to see what words appear in class names. So I wrote up a program to download the source jars for the latest version of all Java libraries available via Maven Central. An hour later I had 60,000 odd jars downloaded on my hard-drive and ready to analyze. Mining Maven central programmatically is interesting enough to deserve an article of its own. Stay tuned for that. I wrote a simple program to then read through the jars and parse all class names into its constituent pieces. I owe the parsing of the names to a clever bit of regex that I found on [Stackoverflow](http://stackoverflow.com/questions/2559759/how-do-i-convert-camelcase-into-human-readable-names-in-java). Its only fitting that the answer was by a user named poly-gene-lubricants!  

So what did I find?

1. It looks like we software engineers are an ego-centric bunch. While I found 2,487 references to "My" in classnames, I only found 4 references to "Your."
2. The most common word was ... Type! A whopping 107,246 times. But that is from a total of 6,906,643 words.
3. We seem to be doing a good job of separating interfaces from implementation as evidenced by the close runner up - Impl - with 102,786 references.
2. We like our fruits - Fruit (18), Apple (133), Berry (36), Banana (12), Peach (2), Mango (35) and Grape (23). And we have some die-hard screaming fans of APPLE (23).
3. We do not like our vegetables. Perhaps those who's mothers fail to impress upon their little minds, the importance of eating vegetables end up as engineers? The first lady may want to take note! Broccoli (0), Spinach (0), Squash (5). Who are these wierdo squash lovers?
4. But we do like meat! Meatball (1), Meat (2), Chicken (58), Beef (2). I shudder to use the Chicken library.   
5. Oh and we like cheese with that (36).
6. Some of us secretly wish we were in the Restaurant business: Cook (19), Waiter (84), Server (17,166), Host (4,772) and Janitor (30). I found no reference to Hostess, proof positive that we do have a gender problem in tech!
7. If we started a zoo, we'd find: Pandas (1), Monkeys (20), Tigers (83), Dogs (132), Cats (351), Snakes (95), Donkeys (5), Zebras (26), Horses (28), Cows (68), Elephants (19), Giraffes (10) and Lions (21).
8. Is there a lot of workplace violence associated with software development? There really ought to be more - Punch (39), Beat (49), Box (4,993), Poke (6) and Kick (20). 
7. Among unusual names, I found:
	1. Valorizer - since nerdy engineer and valor never appear in the same sentence together.
	2. Dateless - surely a reference to the bespectacled geek in the office.
	3. Datefind - the geek's attempt to cure the above stated malady.
	4. Autodate - When the geek resorts to chatting with an IM bot.
	5. Ntpdate - What a geek does to schedule the one accidental date he scores with a girl.
	6. Undated - The result of the one accidental date.
	4. Unblack - pseudonym for Michael Jackson?
	5. Pockuito - A delectable Polish sausage recommended with Prosciutto. 
	5. Sweetener - A class used to make you sign the employment agreement.
	6. Cobbled - A general reference to the way most software is put together. Shame, only one guy was willing to admit it.
	7. Monotonous - Denotes a class with a lot of copy-n-paste!
	8. Seniority - Evidence we don't have an age problem in IT. Oh wait, there was only one reference to it.
	9. Switcheroo - Evidence we didn't pay attention during English class.
	10. Duplciate - More evidence we didn't pay attention during English class.
	11. Boogaloo - A disruptive software solution to the Ghostbusters.
	12. Anonymizing - What the author of Boogaloo did.
	12. Absolutechildren - Remnant of someone's militaristic upbringing.
	13. Delinquency - Unfortunately not of the juvenile (0) type. 
	12. Greenish - a sophisticated word in the male vocabulary for Teal.
	
There are 2,760 class names that are greater than 50 characters in length. Among the largest are:

- "Has This Type Pattern Tried To Sneak In Some Generic Or Parameterized Type Pattern matching Stuff Anywhere Visitor" (from Apache ServiceMix aspectJ 1.8.0)
- "Web Get Number Of Parent Process Instances With Active User And Activity Instance Expected End Date" (from Bonita Serer 5.10.2)

I would have loved to be a fly on the wall when these names were being discussed!

The raw class component names data is available as a [CSV file]({{ site.baseurl }}public/data/class-name-word-frequency.csv).  