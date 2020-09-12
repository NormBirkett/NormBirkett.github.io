---
title: "Intro to Gogetter, Part 0: Intro to the Intro"
layout: post
date: 2020-09-04
categories: gogetter 
comments: true
---

__Summary__:
* Gogetter is a __programming framework__ for complex, data-intensive computing problems.
* It supports a new __programming paradigm__ which has some remarkable properties in the areas of scalability, code quality and debuggability.
* It provides __universal unit-testability__ of all calculations.
* It gives you __universal caching__ of all calculated values.
* So you get all the traditional __performance and correctness benefits__ of caching.
* Plus it supports a new kind of caching benefit that targets the __avoidance of cache misses__, the biggest performance problem in computing today.
* Its prototype exhibits stunningly good __performance__.
* __Code quality__: It makes it hard to write bad code and easy and natural to write good code.
* It shows great promise for promoting __code reuse__.
* There is a natural and stepwise __refactoring path__ into it for legacy codebases.
* And it's __coming soon__, as header-only FOSS, to a C++ development environment near you.

Before you read more about Gogetter, you may want to know:

1. I never set out to create anything like this. I just fell into it. There were some yicky problems in the codebase at work that I wanted to tackle, so I tackled them. Then I noticed that the result was interesting. Most features of Gogetter really just flowed out of the problems I was trying to solve. 
<!--more-->

2. There are some kinds of problems where I am certain that Gogetter is _very_ useful. I suspect Gogetter _might_ be useful on some other kinds of problems. I look forward to learning more about its range of application.

3. Please don't take anything I say about Gogetter as suggesting that it's the Next Big Thing. _It isn't!_ It's a tool that has been useful on a couple of big, complex projects and that shows potential for further usefulness.

5. Gogetter is unlike anything else I've seen, and it's probably unlike anything else you've seen. This makes it a challenging project to describe. Explaining it requires me to talk about matters that computer people aren't used to hearing about, or to talk about familiar matters in unfamiliar ways or in ways that may initially just seem pointless. Paradoxically, for something so novel, the concepts themselves are not deep or intricate; everything looks pretty simple, in fact. I would argue that that very simplicity is itself significant and interesting.

4. The single hardest thing to decide about the project was its name. I have gone through more naming schemes for this thing than you could imagine! In the end, I am grateful to my wife for having made the suggestion that morphed into 'Gogetter' and the rest of the current naming scheme. She had suffered through discussions of many of the rejected names -- itself a big contribution. Gogetter required a novel naming scheme that would emphasize its most central elements _and_ would be easy to remember _and_ would provide a bit of understanding and even guidance to those approaching Gogetter for the first time _and_ would not have confusing echoes of other things in the computing world. Time will tell whether I succeeded!

6. Don't give up prematurely on learning about Gogetter. Try to get as far as the end of Part 3 of the Intro before you conclude that there's nothing here to interest you. It's a difficult project to understand, but _if you face the right kinds of problems for Gogetter_, once you get the hang of it, you'll love it. The prototype has been a stunning success on the two projects on which we've used it, and the other developers who've gotten into it have become enthusiastic about it.

7. Thank you for your interest in Gogetter. It would help me immensely if you would give feedback via the commenting feature, especially of the "I don't get what you're saying about X" variety. (Though the occasional "this is great!" comment will also help with morale.)

So here we go...

[This post revised and expanded 12 Sep 2020]