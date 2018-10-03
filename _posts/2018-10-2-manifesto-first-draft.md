---
layout: post
title: A first draft (slightly improved) of a Manifesto for the (Dynamic Polymorphism) Group
---

_A spectre is haunting our industry -- the spectre of SPECTRE.
But we're not here to talk about that. It just made for the perfect first line of a manifesto. Historical inevitability, you might say.

We're here to talk about Dynamic Polymorphism in C++.

_The history of all Dynamic Polymorphic struggle in C++ hitherto has been the history of class struggle. 
That's one of the things we aim to change.

_Dynamic Polymorphisms of the C++ world, unite! Cast off the chains that bind you! I.e., to the C++ inheritance mechanism.

OK. Fun over. Time to get serious:

We're glad that Static Polymorphism has been getting lots of great support from the language standards in recent years.
The successes of generic programming in C++ demonstrate that that's a good thing.
We applaud and support all such efforts.

But Static Polymorphism isn't the only game in town.
**Dynamic Polymorphism (DP) is also a valuable and powerful tool.
And it is a necessary tool for certain kinds of problems.

The C++ language has built-in support for only one kind of DP -- virtual functions in class inheritance hierarchies.
But the power and value of DP extends far beyond that.
A bunch of us noticed at CppCon2018 that we had all done useful and interesting things involving other kinds of DP.
So we decided to create and sustain a conversation about all kinds of DP in C++ -- beyond just the kind we have in the language now.

##Why is DP important?

Our experience suggests that its value is great, and that it gravitates around three centres:

1. Performance. Vtable lookups, which lie behind virtual-function DP, are expensive. We can do better.
2. Managing and understanding semantic complexity. 
We've found that DP can be useful for managing complex business logic, for decomposing complex legacy code, and for education.
3. Extensibility. At least one of our projects has shown how to extend a class or family of classes from outside.

Some of us are focused one of these, some on more of them. We think all three are areas of great potential and value.

##What do we want to accomplish?

1. Share experience, information, techniques, and insights.
Cross-pollination will help us all solve the problems we're trying to solve.

2. Build better tools that employ DP.
Some of us have been doing this already, but our efforts will be enriched by the cross-pollination of 1.

3. Understand DP better, and help others understand it better.
Some of us are way ahead of others in our grasp on these ideas.
That's a good thing.
But one thing our group needs to produce is exposition of our ideas for the broader, less specialized C++ community.
To teach well requires that we understand better.
And that understanding may then feed back into step 1, with new insights (for example) about the kinds of DP that are possible,
and why one might choose one over another.

4. Recruit others to our group, to gain the benefit of an even wider knowledge base, and to provide our tools and insights
to a broader swathe of the C++ world -- not to mention other language communities.

5. Figure out if we need more support from the C++ language or the STL, and present a united front in seeking that support.
Some of our number are already convinced that we need better support in the language or the standard libraries.
Others of us are not sure.
This will be a point of active debate and discussion.
If and when concrete proposals for the standards committee come out of our efforts, those proposals will be the stronger for
having come through a good winnowing.

##The Dynamic Polymorphism (or whatever we decide to call ourselves) Group
