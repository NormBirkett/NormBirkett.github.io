---
title: "Intro to Gogetter, Part 3: What's Interesting About It"
layout: post
date: 2020-09-10
categories: gogetter 
comments: true
---

So I've claimed that Gogetter is interesting. What makes it interesting? I'll explain here why I find it interesting, and it seems pretty likely to me that you might find one or more of these points interesting, too. Maybe not all of them! That would be perfectly OK.

_Aside: I do_ not _claim that the Gogetter framework/paradigm is suitable for all programming problems. No tool is! I am totally confident, however, based on extensive experience, that it is very well suited to a certain narrow class of extremely hard problems. And it is an open question to me how much more widely its usefulness might extend. I am fairly confident of two things, though. (a) Gogetter's usefulness does extend to more kinds of problems than the one I built it for. (b) Its usefulness does_ not _extend to a really wide class of problems; it is not a competitor to Object-Oriented Programming, for example!_
<!--more-->

I'm going to frame this as a series of observations. In most cases, if you've read Parts 1 and 2, you already know enough to see that my observations are accurate. In a few cases, you'll have to take my word for it until we have more publicly-accessible experience with Gogetter. I will not attempt to distinguish these in the text.

### Observation #1: Gogetter solves a software problem that has been recognized as very important and very difficult. And it solves it very, very easily.

What I mean here is the problem of writing code _that is [unit]-testable_ -- really both unit-testable and integration-testable.

Gogetter's solution is so simple! You can tell a gogetter that you're doing tests, and it will then let you perform insets on calculable gogettums, as thought they were inputs. Then you can perform gogets on dependent values. Instant, universal unit- to integration-testability! And you never have to fit dependency-injection into your code. Never. You can just inject values into your dependency graph anywhere you want, and pull calculated values out for other gogettums -- automatically, built-in, with no thought having to be given to it.

Let that sink in...

### Observation #2: We already have extensive experience with a framework that is similar to Gogetter and has proven very useful indeed.

What framework is that? _The spreadsheet!_ In a spreadsheet, the cell's name ('A12') serves like the gogettum's name in Gogetter. The cell's value may be a literal value (an input to the calculation), or it may be a formula that tells the program how to calculate the value. That formula, of course, is defined in terms of other cells' names. They, too, can contain literal values (inputs), or formulas. And so on. 

My experience has been that in the kinds of calculations that can be conceptually modeled as spreadsheets, the Gogetter framework/paradigm is right at home. Think of these as data-intensive problems where there are a great many different calculated values that are of interest to end users, and where these values form a complex tree-like structure (technically a DAG) with the (many and variegated) inputs as leaves.

Gogetter is in reality more powerful than a spreadsheet in some respects. Each calculation has the full power of a general-purpose programming language for coding it. All C++ types are supported, including user-defined ones, so that one gogettum can have as its value, say, a vector of objects. And there are subtle improvements in the number of ways in which gogettums can depend on each other. (The whatsa for a gogettum can refer to gogettums that can't exist, provided it doesn't actually perform a goget on them.) So it's reasonable to think of Gogetter as a spreadsheeting system that has been transported into a general-purpose programming language and put on steroids; or, in more staid language, a generalization of a spreadsheet. Which doesn't strike me as such a bad thing!

### Observation #3: Gogetter provides caching of all values once calculated.

Caching is a big topic, and it is near and dear to Gogetter's heart.

Consider this: There are several reasons we employ caching, applicable in varying degrees to different systems. They boil down to (a) aiding the cause of *correctness* and (b) improving *performance*. Most discussions of caching that I've encountered focus on the performance benefits, so we'll start there. The two most widely appreciated ways that caching helps performance are these:

* Caching can cut down on repetitive calls to a database or other expensive resource for data.
* Caching of some intermediate values can improve the complexity of some algorithms by orders of magnitude.

In the world of 'data-processing systems', where I have spent much of my career, the first of these benefits of caching is uppermost in developers' minds. In the more computer-sciency precincts of the programming world, the second tends to be paramount. Which makes sense! But the point to grasp here is that one or the other or both of these performance benefits of caching may be relevant to any given system. _And Gogetter gives you both benefits of caching pretty much automatically._ You can even cachify data simply by defining it in a gogetter-system and then performing gogets on it. That pulls it into the gogetter's value cache, and there it will sit until you destroy the gogetter. Easy-peasy caching!

In my opinion, the bigger benefits of caching lie in the realm of improved correctness. But these benefits are more diffuse, so they are less widely appreciated. How does caching aid correctness?

* Caching protects intermediate values from state changes (assuming each item in the cache is immutable, which is usually true and is very much the natural way to program in Gogetter).
* Caching allows you to shrink the length of a database-read transaction window, which reduces the chances of an inconsistent data snapshot corrupting a calculation. This makes it less potentially disastrous to avoid using transaction-management on read-only transactions on relatively static data. Many 'data-processing' systems in fact don't bother with transaction management on reads for 'fairly static' data. Caching helps them get away with this more of the time. And an aside: There may be a pretty big performance benefit hiding in this point, in addition to the correctness benefit.

I think these correctness benefits of caching deserve to be more widely appreciated. Especially the first, which can help us prevail in the war against state that takes up so much of our time and energy. In fact, the functional programming community does talk about this benefit; they just don't tend to use the word 'caching', preferring for example 'memoization'.

Anyway, Gogetter more or less throws in caching of all gogettum values for free. Which can provide all these benefits of caching if you design your gogetter-systems carefully.

What is arguably even better is that Gogetter furnishes a new kind of caching benefit! What is that? I was _hoping_ you would ask!

### Observation #4: Gogetter lets us do caching so that we avoid the biggest performance problem in contemporary computing.

If you look at today's discussions of high-performance computing, you quickly find out that the Big Bad Problem is memory-cache misses. This arises from the 'Von Neumann bottleneck' -- the fact that the CPU has to read data from and write data to memory, and by the standards of the things that go on inside CPUs, access to memory is slow. So the CPU has its own cache, a hardware cache with complicated logic governing it. Or even a hierarchy of two, three, or four hardware caches. It turns out that retrieving a piece of memory that isn't in the memory cache takes something like 10 to 100 times as much time as retrieving memory that is cached. When this happens, we call it a cache miss. And that's the big challenge for writing high-performance software nowadays.

Gogetter -- this happened entirely by accident -- provides a funny little bit of help in this area, in a way I've not seen anywhere else. Because we want to avoid cache misses, the name of the game is 'locality of reference'. That is, try to reference memory from the same neighborhood before you move on to a different neighborhood and access that, and so on. The reason is that memory locations near each other have a decent probability of being pulled into the memory cache together, so the access to the first one causes the second one to be a cache hit. The point is, don't go hopping from one address to another one far away. One thing that can help, then, is to reduce the number of addresses we need to access. When we access fewer addresses, we'll have fewer cache misses.

So it would be nice if we could use caching (that is, software caching, as opposed to the memory-hardware kind) to avoid memory accesses. How might this work? Suppose you're going to do something with the values stored at memory addresses a, b, c and d. What you're going to do with them will involve stirring them together and boiling them down to a new value e'. Suppose further that most times you need a, b, c or d, you end up boiling them down to e'. So let's just calculate e' once and save (cache) its value at address e. Surely then we can avoid all those accesses to a, b, c and d! Well, yes and no. That is, yes -- but with some trickiness.

Let's think about the ways value-senses appear in code. There are traditionally two. Gogetter adds a third, the goget.

#### Value-senses manifested as variables

A value often manifests itself in code as a variable -- meaning any object, whether mutable or immutable, that can acquire and retain a value.

How can we cut down on memory accesses when our values are stored in variables? By organizing the arrangement of variables in memory so as to optimize access patterns, usually by putting values that tend to be accessed near each other in time, near each other in memory. Which of course is much easier said than done! This exercise goes by the name of 'data-oriented programming', and it can be effective, though to my knowledge no generalized methods or algorithms are widely accepted in this area. One pattern-choice has been noticed, which is AOS versus SOA -- Array of Structures versus Structure of Arrays. But that seems to be the state and extent of the art.

I'll note in passing that Gogetter brings all of the really important values into one managed collection, where in theory some kind of localization algorithms could be tried out. I don't know of any other framework, system, or paradigm that achieves this. Of course, I don't know if Gogetter can do anything successful along these lines. For now it remains an intriguing possibility, no more. But if it were to work out, it would mean that Gogetter represents a workable and generalized implementation of data-oriented programming.

#### Value-senses manifested as function calls

Another way values show up in code is as function calls. We've seen this in an earlier post in this series:

```c++
  finance_charge(average_daily_balance, interest_rate, days_in_stmt_period)
```

How can we cut down on memory access here? I have seen no general techniques for this. The usual technique for cutting down on machine cycles -- and as far as I know the best available technique -- is memoization. That is, inside finance_charge(), we would keep a collection of all the finance charges we've previously computed during this run of the program, keyed by their arguments' values. When a new call comes in, we check whether that set of argument values has been encountered before. If so, we read the value from our collection and return it without recalculating it; if not, we calculate it and save it in the collection before returning it.

If the calculation of the finance charge is a CPU-intensive operation, memoization will be useful. _It doesn't help us avoid memory access, however._ That's because the caller of finance_charge() has to access average_daily_balance, interest_rate and days_in_stmt_period -- wherever they happen to be in memory -- before we can call our function and execute our memoization lookup step. Furthermore, functions are often called in a variety of ways in code, some of which yield usefully memoized values and some of which don't. So it is possible, even likely, that applying memoization to a function that is called a lot will create the need for a large memoization collection, most of which is never reused. Which could make matters *worse* for memory access.

#### Value-senses manifested as gogets

In a goget for the finance charge, such as:

```c++
  auto fin_chg_v <<= stmt.goget(finance_charge)
```

all we need to access is the gogettum's name (which is constexpr), and then gogetter can figure out whether it already has its value in the value cache. If it does, it avoids all reference to the inputs to the finance charge calculation.

This is caching to avoid memory access! And remember: Memory access is the biggest performance bottleneck for most software these days. So I think this is interesting. And it might be part of the explanation for my next observation.

### Observation #5: Gogetter's prototype is surprisingly performant.

When I built the prototype and eliminated one or two dumb implementation mistakes from it that caused significant performance problems, I was amazed at how fast it was. I have not gotten to the hard work of really optimizing the prototype. I am aware of several optimizations that should be useful, but I've only done a couple that were pretty obvious. _But a very large calculation implemented in the prototype of Gogetter performs as well as a reasonably fast predecessor implementation that doesn't have a software framework involved in organizing the calculations._ I was surprised that I had gotten such good performance with little attention paid to optimization. I was definitely *not* expecting that! Usually, when you rip out the guts of a huge calculation and rebuild them with a new software framework right in the middle of all of the calculations, you expect to have to do a _lot_ of optimization work to get it to perform acceptably. I had built Gogetter motivated by correctness and code-clarity concerns, and it performed as well as non-framework code just a small step out of the box!

So the obvous question is, Why? My hypothesis is that two features of Gogetter explain this.

1. The strictly lazy evaluation plus universal caching has actually cut down on the number of calculation steps taking place.
2. The memory-access-avoidance outlined under the previous observation may also be playing a role. 

To be fair, I have not yet done sufficient benchmarking to be sure of this hypothesis. And performance hypotheses are notoriously unreliable until you do really thorough benchmarking and profiling. So doubters would have a point here -- though I must admit I would be curious what theory they might advance!

But whatever the reason, it is a good thing. And it shows promise that once Gogetter is built and _really_ optimized, it could sing.

### Observation #6: Gogetter supports a new and powerful kind of polymorphism.

Recall our original whatsa for finance_charge:

```c++
whatsa finance_charge (
        money_amt_t calc (
                let avg_bal <<= average_daily_balance;
                let int_rate <<= interest_rate;
                let days <<= days_in_period;

                << avg_bal * int_rate * days / 365.0;
        )
)
```

Now imagine how it might have evolved after several years of new features being added to the system.

```c++
whatsa finance_charge (
        money_amt_t calc (
                let interest_chg_bal <<= interest_charging_balance_id; // Assume we've defined this.
                money_amt_t balance{0.0};
                switch(interest_chg_bal) {
                case INTEREST_CHG_BAL::AVERAGE:
                        balance <<= average_daily_balance;
                        break;
                case INTEREST_CHG_BAL::MAX:
                        balance <<= maximum_daily_balance; // Assume this has been defined, too.
                        break;
                case INTEREST_CHG_BAL::FIXED:
                        balance <<= fixed_charging_balance; // And this.
                        break;
                }
                let int_rate <<= interest_rate;
                let days <<= days_in_period;

                << balance * int_rate * days / 365.0;
        )
)
```

Now we've got a finance_charge that is polymorphic -- in the somewhat strange sense in which the computer industry uses that term. That is, we refer to it by one name, but the way it is calculated varies based on one of the terms of the credit-card account.

Notice a few things about this:

* The inputs to the calculation of the one finance_charge vary. For AVERAGE and MAX, of course, the ultimate inputs will be the same (namely all the daily balances), but for FIXED, not so.
* At run time, only the required input is accessed and (if not required elsewhere) calculated. This can accentuate the performance benefits of using Gogetter, because in a large, complicated graph of gogettum interdependencies, we only explore the minimal subgraph required by the actual run-time requirements of the calculation (assuming we've programmed gogetter system wisely).
* This sidesteps two of the problems of traditional OO class-based polymorphism: (1) The signatures of the multiple versions of, say, finance_charge, can be radically different. No matter. That's because the identification of inputs has been moved out of a function's argument list into the 'inside' of the whatsa defining the result. (2) There is no vtable to impart inefficiency to calls to the calculation.
* On this last point, I would go even further, and say that this polymorphism appears to be theoretically optimal. Why? The calculation must obtain the interest_charging_balance_id in order to determine what else it needs, but after that it retrieves what it needs and only what it needs. So the polymorphism here is optimally efficient. _It relies only on access to the minimum set of data for the actual run-time requirement._ I don't see how it or any other approach could do any better than that.

I think of this as 'value-sense polymorphism'. We identify a value by its sense -- by some kind of description of its role and meaning in the calculation -- and then we can make it calculate itself in different ways if that is required by the problem we're solving. Period. That's it. No complications! And in fact no overheads for this flexibility. This strikes me as pretty close to an ideal kind of polymorphism for value calculations.

### Observation #7: Experience has shown that the Gogetter framework makes it easy to write good code and hard to write bad code, on at least a couple of goodness-badness metrics.

In Gogetter, it is natural to write small, single-meaning gogettums. It is hard (though not impossible) to write big, overcomplicated, multiple-responsibility calculations. And as mentioned already, each gogettum is instantly unit-testable -- no thought required, no dependency injection required, etc. I think a programming paradigm in which it is natural and easy to write small, well-defined pieces of code that are easily unit-testable, and positively difficult to deviate from this, is kinda cool!

It is possible to be absolutely fanatical about the DRY principle in goget. In fact, it's easy. Remember that whatsas can call functions, or even construct objects and employ them. So if two whatsas have too much in common, you have the full usual range of tools to pull out the commonality into a separate, single location in your code.

I believe it will be possible and desirable to extend Gogetter into generic programming by writing templated whatsas. You can class this as one of the future research goals of the project. In fact, it should be possible to write templates that take in multiple whatsas, to reflect little common subgraphs that show up in different places. I've already seen several real-world examples of this.

If you go through the SOLID principles, you will find that Gogetter stands up pretty well. Briefly:
* Single-Responsibility: Each gogettum is a value -- a value-sense that implies a value-referent for each calculation instance, which is hopefully matched by the actual value-wannabe computed. The whatsa for each gogettum tells the gogetter how to compute the value. Those seem like pretty unified, single responsibilities!
* Open-Closed: If you do a good job of conceptually analyzing your value dependencies, you find that gogettums are mostly closed for modification, and tend to stay that way. And the whole gogetter system is obviously quite open to extension. I'm not sure how to quantify this, but based on my experience, gogettums are far more closed for modification than units of procedural or OO code tend to be, though perhaps if I found a system with procedural/OO code that consisted entirely of small, well-defined classes and functions, this advantage would vanish.
* Liskov Substitution: This one is sidestepped. There is no inheritance. And the polymorphism is just variations in how a particular gogettum might need to be calculated.
* Interface Segregation: Every gogettum can be retrieved individually, by name, by client code. So the overall calculation (represented by the gogetter system) lets clients pick and choose what data they want, and it even encourages them to goget only what they need, because that way they won't pay for calculating what they don't ask for -- the central virtue of laziness (in computing).
* Dependency Inversion: Most of this one as it pertains to the OO world is largely irrelevant to Gogetter, but I would argue that the gogettum represents about as pure an abstraction as you can find. You give a meaningful (it is hoped) name to a value that is defined by how it is calculated from the other named values, and then you ask for it if you end up needing it. That is a level of semantic purity I haven't seen too many other places.

### Observation #8: Gogetter encourages you to think about value calculations in terms of graphs, which is theoretically powerful.

It struck me one day when I was starting to work on what would become Gogetter that there's a powerful fact about computer value calculations that we rarely hear about (at least, I've never heard anyone say this):

>  __Every value that has *ever* been calculated on any computer, or that *will* ever be calculated, was the root node of a finite DAG (Directed Acyclic Graph), where all the other interior nodes were intermediate values that were calculated en route to the root (!), and all the leaf nodes were inputs to the calculation.__

That strikes me as a pretty powerful and useful observation, and as quite obviously true.

Now it was good news to me, because I happen to be quite fond of DAGs. I guess I'm slightly obsessed with them! I blame it on my mother, for having taken me to play in various banyan trees when I was a child. I *loved* banyan trees, and they fascinated me. They are the arboreal implementation of DAGs. Anyway, I'll seek help sometime.

But meanwhile, I also like this insight because it identifies a powerful commonality among a vast set of involved and quite disparate computations. And I also think it is quite a useful mental tool for figuring out how to break down value calculations.

It also strikes me as potentially powerful for developing visualization and debugging tools for Gogetter. I've already worked on these problems a bit, and the possibilities are quite exciting (to someone with DAGomania, at any rate).

In the real world, the abstract, value-sense DAGs have all kinds of conditionality inside them, which can make them slightly ill behaved. But every actual instance of the calculation will involve resolving all the conditions and getting one very straightfoward DAG out of the process. That way of thinking about the process of carrying out value calculations, especially the horribly convoluted ones that I seem to work on a lot, has struck me as mathematically beautiful, and again, quite clarifying as to the nature of what is going on.

By now, it should be obvious that this way of thinking is very natural in Gogetter. I'd even say it's required for using Gogetter effectively. DAGophobes need not apply...

### Observation #9: Gogetter is surprisingly easy to debug.

I've found that a few tools are helpful for this purpose, but they proved to be easy to write. And the debugging process itself is really pretty easy -- though of course when your gogettum DAG is sufficiently convoluted, you may have to stop at many gogettum/whatsas in a given calculation.

Once we've got a working Gogetter system in this open-source project, I will start to document this aspect of it more fulsomely, with some working examples to study.

### Observation #10: Gogetter comes with a remarkably simple refactoring pathway into it (or out of it).

Provided you can get some kind of automated regression tests over a body of legacy code, then you can refactor that code. (I am well aware -- painfully aware -- that you can't always do that, or at any rate can't always do it within reasonable cost constraints.)

The best kinds of refactorings are simple ones involving small incremental changes to the code. 'Extract method' is a good example of this. In one common use case, you start with an overly long member function, and you take a piece of its code and move it into a separate method, putting a call to the new method where the code used to be. And it's amazing how much you can accomplish by repeated application of this kind of simple, well-defined refactoring step.

Refactoring code into Gogetter can be like this if you let it. And refactoring code inside Gogetter is strikingly easy.

In fact, there are only four logically-possible refactorings involving Gogetter!

* __Gogetify__
* __Split__ [gogettum]
* __Merge__ [gogettums]
* __Ungogetify__

Each of these has a version that you apply around an input gogettum and another that you apply around a calculable gogettum. We'll look at the input versions first.

#### Refactorings on input gogettums

* Input gogetify
  * You have an input gogettum _I_, and some code somewhere (outside Gogetter) that computes it.
  * You redefine _I_ as a calculable gogettum.
  * And you move the code that computes it into its whatsa.
  * Plus you define new input gogettums for its inputs, unless they happen already to be in your gogetter system.
  * And you add code somewhere to inset any new input gogettums.
* Input split
  * You start with an input gogettum _I_, which is some sort of compound/aggregate object.
  * You define in its place input gogettums _I1_ and _I2_, each of a type that includes some of the data in _I_'s type.
  * You replace your code that does the inset on _I_ with code that does insets on _I1_ and _I2_.
  * You replace every goget on _I_ (whether internal or external) with a goget on _I1_ or _I2_ or gogets on both.
* Input merge
  * This is pretty close to the exact reverse of splitting an input gogettum, so I'll leave it as an exercise to the reader.
* Input ungogetify
  * You have an input gogettum _I_ that you want to remove from your gogetter system.
  * If there are internal gogets on _I_, you will have to turn the gogettums that perform them into input gogettums.
  * If there are external gogets on _I_, you will have to convert them to references to wherever _I_ will now be found in your codebase.
  * Now you can delete your inset on _I_.
  * And delete _I_'s whatsa.

#### Refactorings on calculated gogettums

* Calculable gogetify
  * You have some code that gogets value _G_ from a gogetter, and uses it to compute _F_.
  * So you define _F_ as a calculable gogettum, and move its calculation code into its whatsa; _G_, naturally, will be one of its inputs.
  * And now you obtain _F_ by gogetting it from the same gogetter.
* Calculable split
  * You have a calculable gogettum _C_.
  * You break it into smaller gogettums _C1_, _C2_, and _C3_, say.
  * Let's say _C1_ depends on _C2_ and _C3_, and _C2_ and _C3_ between them use all of _C_'s inputs.
  * Now you can write other code that uses only _C2_, for example.
* Calculable merge
  * You start with calculable gogettum _C_ that depends on _D_ and _E_ (both calculable).
  * You define a new _CDE_ whose value equals _C_'s and whose inputs are the union of _D_'s and _E_'s inputs.
  * The whatsa for _CDE_ is a mashup of _C_'s, _D_'s and _E_'s whatsas.
  * And you eliminate _D_ and _E_.
* Calculable ungogetify
  * You start with calculable gogettum _C_, which you want to move out of your gogetter system.
  * Let's assume for the moment that no other gogettums perform (internal) gogets on _C_. If they do, then this refactoring is going to be more complicated, as you will need to turn _C_ into an input gogettum, and you might get tangled up in some tricky sequencing problems. (If you want to take _C_ out of your gogetter system, it is in fact unlikely that other parts of that system depend on it.)
  * You move _C_'s whatsa code out into some non-Gogetter code context.
  * Note that the gogets in that code just become external gogets on the same gogetter that was calculating _C_. They shouldn't have to change very much.
  * Any external gogets on _C_ now just have to obtain _C_ from wherever you've now put it.

I don't recall ever seeing a use for ungogetification, though I have thought that if you ever wanted to stop using Gogetter in a piece of code, you would achieve this through a sequence of ungogetifications. So far, however, I've not encountered any reasons to want to escape from Gogetter! Quite the opposite.

Merging has also been extremely rare in my experience, whereas splitting is quite common. I guess it's rare that we find that we made our pieces of code too small and single-purpose!

Normally you should have unit tests to modify, or to create, in conjunction with each of the refactorings on calculable gogettums, but I've not chosen to spell those changes out, lest I wax tedious.

Particularly by means of the gogetification refactorings, it should be possible to gradually have a Gogetter structure grow and absorb a large body of legacy code. I am experimenting with this on Gogetter's prototype in fits and starts at my day job.

### Observation #11: Gogetter blurs the line between value semantics and reference semantics in an interesting way.

I believe there are other techniques out there that do this (e.g. handles), so I don't claim any uniqueness for Gogetter here. But this point is interesting nonetheless.

Consider the goget. It behaves in many ways like an assignment from a reference type, but with a crucial difference: The goget mechanism of the gogetter manages the existence of the thing 'pointed to'! So if it exists already, great! Just return it. If not, well then go compute it and then return it.

This of course is why the goget is called a goget. The gogetter goes and gets the value. And it knows how to get it if there is any way that it can be got. It also manages its location in memory, keeping any other gogettum from overwriting it.

So if you have lots of values you need to get out, you have to keep track of only one reference (your reference to the gogetter), and then you get nice robust referenceability of all the gogettums. In other words, you've contained the reference semantics to one thing and gained reliable availability, traditionally associated with value semantics, on a whole bunch of things.

I find that interesting.

### Observation #12: Gogetter gives rise to a new and interesting programming paradigm.

This observation overlaps a lot with some of the earlier ones, but it highlights an additional point that I think is important.

When you write code to go into Gogetter, you quickly find that you're doing something very different from classic structured-procedural or OO coding. It makes you think differently -- radically so. In other words, it has you programming in a different _paradigm_ -- that is, in a different thought-world. 

My name for this new paradigm is __Value-DAG Programming (VDP)__. I will have a lot more to say about it, but for now, here's a preview:

* VDP is a kind of __declarative programming__.
* But it scales extremely well as to performance, debugging, and programmer-understanding -- though the last of these is not without its challenges. _These are features not usually associated with any declarative paradigm or language._
* Most declarative languages and paradigms are special-purpose (SQL, e.g.) But VDP is general-purpose. In that respect, it resembles the __functional programming (FP)__ paradigm, which happens to be another declarative paradigm.
* But unlike FP, it scales well -- see two bullets ago. Really well. And it works well on extremely complex data, which is not a traditional strength of FP.
* VDP also shows promise to be a very manageable form of __data-oriented programming (DOP)__, as mentioned above. I would go so far as to say that VDP bids fair to become the first general-purpose, usable implementation of DOP. If so, then it can transform DOP from its current status as a set of ideas that guide people working in other paradigms into a real programming paradigm of its own. I know that's a pretty big claim, but I stand behind it.

People like me who are into programming paradigms should find these claims interesting if I can make them come true. Watch this space!

### Conclusion

I doubt that any one person will find all my observations here to be equally compelling. But I hope at least some people will find a few of them intriguing. Please talk back in the Comments section! I would love to hear how these points strike you.

And I hope I've piqued your curiosity about Gogetter at least a little tiny bit.

Next and final stop for this intro: What is to be done?

[This post revised and expanded 12 Sep 2020]