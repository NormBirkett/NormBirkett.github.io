---
title: "Intro to Gogetter, Part 4: What Is To Be Done?"
layout: post
date: 2020-09-12
categories: gogetter 
comments: true
---

Here is the situation:

1. A prototype of Gogetter has been built (and rebuilt once) and has been in heavy production use for several years, with two major projects built on top of it.
2. That prototype is proprietary.
3. I've learned so much from building the prototype that it seems best just to rebuild it from scratch as open-source. It won't bear much resemblance to the proprietary prototypes.

What are the pieces that need to be built in the Gogetter project? (Not necessarily in order.)
<!--more-->

> __A.__ The __syntax__ for defining and using gogettums laid out in this Introduction is slightly utopian. I don't believe it will be possible to preserve it exactly as-is in C++. So the C++ syntax needs to be worked out. This has two parts:

> __A1.__ The __EDSL__ for defining gogettums (i.e., the whatsa syntax), including the syntax of internal gogets.

> __A2.__ The __interface__ for insets and external gogets.

> __B.__ The gogetter itself -- meaning the __run-time component__. 

Three important and tricky pieces that will play important roles in *all* of the above are:

> __AB1.__ The name system

> __AB2.__ The gogetter-to-whatsa run-time interface 

> __AB3.__ The specialized variant type that will contain all gogettum values. This will provide built-in support for many basic types, and also allow extension by users for their own types.

Note: I intend to provide A and B to potential users in __header-only__ form, and am pretty sure this will be possible.

> __C.__ __Unit tests__ for Gogetter itself (i.e., for all of the above). It is my intention to build all or most parts using __TDD__.

> __D.__ Extended and worked-out __examples__ of using Gogetter. These need to cover the varieties of DAG gestalts. So far I have identified three good candidates and some maybe-good candidates. I'll mention the good candidates here:

> __D1.__ A revolving credit calculator that could be used by a bank

> __D2.__ A limited-purpose XML parser (for systems that use XML for data structures)

> __D3.__ A cricket match analyzer

These are meant to serve several purposes:
                
1. As __documentation-by-example__ to those who are interested in learning how to use Gogetter
2. As research into how well Gogetter serves __differently-shaped calculations__
3. As __validations of the syntax__ defined in A1 and A2, to make sure it is adequate for reasonably (but typically) complex projects

There are a couple of advanced features I want to build next:

> __E.__ Gogets with __quantification__, whether universal or existential

> __F.__ Enforcement of various __subscript system shapes__: sparse, growing, etc.

> __G.__ A mechanism for defining __subscript system converters__

And then there are several longer-term questions I hope to pursue after all of that:

* It should be possible to add __parallelization__ to the gogetter. How well would this work?
* The primary use-case for Gogetter is on immutable value-structures. But can Gogetter be extended to serve highly __stateful calculations__? I know it can work on cases with very limited state, and that has been quite useful.
* Can the values of gogettums be streams, with long-running gogettum calculations consuming or producing them? If so, we would have a truly push-based OR pull-based __reactive programming__ framework.

Is Gogetter the long-sought "one tool to bring them all and in the darkness bind them"? I hope not! That sounds awful! But is it a tool that deserves to see the light of day and to be tried out on various problems until we can figure out how to tell which problems it will solve well? I think so -- from all that I've seen of it while building its prototype and using it pretty heavily in real-world code (and teaching other developers to use it).

I hope it will interest you enough to follow its progress, and even to find ways to help out, if you are so inclined.

Thank you for reading about it! Please leave comments, especially questions.

Note: The project will use __C++17__ for now. Once I figure out the github issues, I want to validate the code along the way in all the major C++ compilers, though I will be doing my own work using MSVC.
