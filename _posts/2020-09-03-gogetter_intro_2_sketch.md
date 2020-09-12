---
title: "Intro to Gogetter, Part 2: A Quick Sketch of How It Works"
layout: post
date: 2020-09-07
categories: gogetter 
comments: true
---

### Just the finance charge

Here's how we would code the calculation of our finance charge in Gogetter. For this first go-round, we'll leave out the other parts of our credit-card statement. The syntax here is an idealized Gogetter syntax that is only pseudocode at this point in time; I'm sure it will take some tweaks to make it realizable in C++.
<!--more-->

```c++
gogetter_system statement () {

        types (
                date_span_t
                int_rate_t
                money_amt_t
        )

        whatsa days_in_period (
                date_span_t input 
        )

        whatsa interest_rate (
                int_rate_t input 
        )

        whatsa average_daily_balance (
                money_amt_t input 
        )

        whatsa finance_charge (
                money_amt_t calc (
                        let avg_bal <<= average_daily_balance;
                        let int_rate <<= interest_rate;
                        let days <<= days_in_period;

                        << avg_bal * int_rate * days / 365.0;
                )
        )
}
```

I'll explain this code more fully below, but, in a nutshell, it defines to Gogetter a single calculation -- of finance charge -- in terms of three inputs. You can think of the code above as living inside Gogetter, though it is in fact client code. The general term for what is defined here is a __gogetter-system__. That's 'system' in a logical-mathematical sense, an interconnected system of value-senses, some of which are inputs while the others are calculable in terms of the inputs.

For Gogetter to _do something_ with this information, we need some outside client code -- meaning client code that sits outside Gogetter. There are three steps this outside code must perform.
* First, it must create a gogetter object that is tied to the definition above of 'statement'. __Important note: We will use 'Gogetter' with a capital G to refer to the project and the totality, and 'gogetter' with a little g for the gogetter object created in code.__
* Then it must provide the necessary inputs to this gogetter for the calculation it will ask it to perform.
* Finally, it can ask it for calculated values.
These steps are usually widely separated in code, though they don't have to be. I'll illustrate them as separate snippets, though I will assume I have a non-const reference to the gogetter in the second and third snippets.

First, we create a gogetter:

```c++
gogetter stmt(statement);
```

The 'statement' inside the parenthesis tells the gogetter what gogetter-system it is to implement.

Second, we supply inputs:

```c++
stmt.inset(days_in_period) << some_date_span;
stmt.inset(interest_rate) << some_interest_rate;
stmt.inset(average_daily_balance) << some_balance;
```

What happens here is pretty straightforward: We use a construct called an __inset__. This takes two very different kinds of parameters. The first is a name that has been defined within our gogetter-system. That tells the gogetter which value-sense's value-referent is being supplied. The second, shown after the '<<', is the value-referent to be assigned to that value-sense for this calculation.

In production contexts, insets can only be performed on names that are declared as 'input' in the gogetter-system.

Finally, we harvest outputs -- or, in this case, the output, since our gogetter-system defines only one calculable value-sense:

```c++
// Here are several ways of retrieving the finance charge:
auto fin_chg_v <<= stmt.goget(finance_charge); // Initializer of an 'auto' declaration
money_amt_t fin_chg_v_again <<= stmt.goget(finance_charge); // Initializer of exlicitly typed declaration
const money_amt_t & fin_chg_ref &= stmt.goget(finance_charge); // Initializer of a const ref
const money_amt_t * fin_chg_ptr *= stmt.goget(finance_charge); // Initializer of a const pointer

money_amt_t fin_chg_assign_to;
fin_chg_assign_to <<= stmt.goget(finance_charge); // Assignment to a correctly typed variable

const money_amt_t* fin_chg_ptr_assign_to;
fin_chg_ptr_assign_to *= stmt.goget(finance_charge); // Assigning to a const pointer

// One other small point: You can retrieve inputs as well, not just calculables.
int_rate_t irate <<= stmt.goget(interest_rate); // Refs and ptrs and so on are also available.
```

The construct used in all these cases is the __goget__, which behaves much like a regular initializer or assignment, except that it has an extra element tacked on -- namely (!) the name of a value-sense defined in the gogetter-system. This can be an input or calculable value-sense, it matters not.

Note also that there are both __inner__ and __outer gogets__. The inner gogets are found inside the definitions of calculable values inside the gogetter-system. The outer ones are found in outside client code that asks for values from the gogetter.

__One very important feature of Gogetter is that it is fanatically lazy.__ No storage space inside the gogetter is allocated for any value until it is needed, whether because the value was supplied as an input or because a calculable value was asked for. Obviously, then, the value is also not calculated until that moment. This turns out to be a valuable optimization feature.

__But equally important, its laziness means that once a value _has_ materialized inside the gogetter, that value is immutable.__ This feature turns out to be important for a number of reasons. We'll come back to this later. (There are certain rare and controllable exceptions, but by default the values are left alone after they are computed.)

Gogetter, like C++ in general, supports value semantics. And just as in C++, you can abuse those value semantics by declaring pointer-typed variables and storing addresses in them. Thus, just as in C++, you are urged to use the value semantics if you want to live a long and happy life, but are permitted to deviate from this salutary course when you judge that you must. Which you inevitably will, this being C++! And then you, as always with C++, will have the rest of your tenure with that codebase to regret your decision.

In other words, Gogetter provides a very C++ish working environment: You are given the tools to do safe and sensible software development, but you are also given the tools to do high-risk, crazy things in the pursuit of performance objectives or just plain self-destructive thrill-seeking behavior. As <a href="https://twitter.com/TimSweeneyEpic/status/1195194522843729921">Tim Sweeney</a> has said of C++, so Gogetter "gives you enough rope to shoot yourself in the foot with a can of worms".

One area where Gogetter improves on C++ a bit is in having constness on by default. It does allow you to override this default if you think you know what you're doing. _Caveat emptor_ and all of that.

### Scaling up

Now let's see the gogetter-system for our full-blown, from-the-ground-up calculation of finance charge. This will provide hints of some the richer capabilities of Gogetter, though I don't intend to dig too deeply into those at this point.

```c++
gogetter statement () {

        types (
                date_range_t
                date_span_t
                int_rate_src_t
                int_rate_t
                money_amt_t
                transaction_t
        )

        whatsa date_range (
                date_range_t input
        )

        whatsa interest_rate_source (
                int_rate_src_t input
        )

        whatsa starting_balance (
                money_amt_t input
        )

        whatsa [charge_index] (
                const consecutive
        )

        whatsa charges[charge_index] (
                transaction_t input
        )

        whatsa [cash_advance_index] (
                const consecutive
        )

        whatsa cash_advances[cash_advance_index] (
                transaction_t input
        )

        whatsa [payment_index] (
                const consecutive
        )

        whatsa payments[payment_index] (
                transaction_t input
        )

        whatsa [other_credits_index] (
                const consecutive
        )

        whatsa other_credits[other_credits_index] (
                transaction_t input
        )

        whatsa days_in_period (
                date_span_t calc (
                        let dt_range <<= date_range;
                        << dt_range.end() - dt_range.begin();
                )
        )

        whatsa interest_rate (
                int_rate_t calc (
                        let dt_range <<= date_range;
                        let rate_src <<= interest_rate_source;
                        << rate_src.interest_rate_during(dt_range);
                )
        )

        whatsa average_daily_balance (
                money_amt_t calc (
                        let start_bal <<= starting_balance;
                        let charges_r &= charges;
                        let cash_advances_r &= cash_advances;
                        let payments_r &= payments;
                        let other_credits_r &= other_credits;
                        let dt_range <<= date_range;

                        auto transactions{ 
                                merge(charges_r, cash_advances_r, payments_r, other_credits_r) 
                        };

                        << avg_daily_balance(start_bal, transactions, dt_range);
                )
        )

        whatsa finance_charge (
                money_amt_t calc (
                        let avg_bal <<= average_daily_balance;
                        let int_rate <<= interest_rate;
                        let days <<= days_in_period;

                        << avg_bal * int_rate * days / 365.0;
                )
        )
}
```

A few comments on this:

* Each value-sense that is defined in a gogetter-system is called a __gogettum__ (plural __gogettums__). 
* The value-referent and value-wannabe of a gogettum are referred to as the __gogettum value__. When we need to make clear that we mean the value-referent, we can speak of the __correct__ or __right value__. For the value-wannabe, we usually just say something like, 'Here's what it came up with.'
* The code objects that define gogettums are __whatsas__. The idea behind this name is that when you say to the gogetter, 'Go get me the finance charge', its first thought is, 'What's a finance charge?' So it ... well, you get the point.
* Some gogettums (in fact, most, if the projects I've seen are anything to go by) have __subscripted names__. There are profundities here that we must skate past for now. Suffice it to say that a gogettum's name can have several subscripts. I've not yet seen a need for more than three, but I'm hoping that Gogetter can make this unlimited. We'll see.
* There are deep subtleties around the __subscript types__ or __dimensions__ that are defined here. They are the glue that holds together __subscript systems__, which are groups of names that are subscripted in sync to refer to the same entity. If you are a Data-Oriented Programming fan, and you think you smell Structures of Arrays (SOAs) here, then you have a good nose.

We'll skip the declaration of the gogetter object, as that is unchanged from the first section of this post, and go straight to the insetting of inputs:

```c++
stmt.inset(date_range) << some_statement_date_range;
stmt.inset(interest_rate_source) << some_interest_rate_source_object;
stmt.inset(starting_balance) << some_starting_balance;

for (size_t i = 0; i < some_collection_of_charges.size(); ++i) {
        stmt.inset(charges[i]) << some_collection_of_charges[i];
}
stmt.close(charges);

// ... and so for the other collections of transactions
```

Retrieval via gogets is unchanged from before, except of course that you have a lot more choices of gogettums to ask for. So we won't bother with code for that.

A few more comments:

* I didn't mention it before, but the input value to an inset can be any rvalue of suitable type.
* As we've seen already, so here: Names can have dimensions. The name 'charges[i]', for example, has one dimension. That is, each specific gogettum that will get the name 'charges' will be distinguished by one subscript.
* When 'charges' was defined (in a 'whatsa', earlier), it was given the dimension 'charges_index', which is defined as 'const'. That means the gogetter needs to be told when all the 'charges[i]' values have all been supplied, so it can close the dimension and keep it constant for the duration of this gogetting (which is what we call one instance of our grand calculation).

Something more needs to be said at some point about what actually happens when a goget is executed. Here are the steps:

  * The gogetter looks at the name of the gogettum.
  * It checks whether it has already computed that gogettum.
  * If it has, it just returns the value that is already there in its cache.
  * If not, it checks that it has a whatsa for the gogettum.
  * If it doesn't, it signals an error. (How it does this is a question we will defer.)
  * If it does, but the whatsa indicates it's an input gogettum, it signals an error (ditto).
  * If we've reached this point, the gogetter sets about computing the gogettum.
  * This it does (obviously) by executing the instructions in the whatsa.
  * Note that a whatsa can itself contain gogets that request the values of other gogettums.
  * These are executed in a logically recursive manner. (Implementation is another matter, but we will defer discussion of that.)

### The big picture

Out of all of this there emerges an ontology, if you will. Let's summarize it, mostly by way of review.

* A computation -- that is, an instance or occurrence or execution of a computation -- is called a __gogetting__. 
* The (client) code that specifies all the values to be used and/or computed in a particular kind of gogetting is called a __gogetter-system__. 
* The __gogetter__ is the run-time component of this thing, which performs all the operations involved in a gogetting by referring to the gogetter-system.
* Each value that we might want to compute in a particular computation is represented as a __gogettum__, of which the plural is __gogettums__. The gogettum is an abstraction of a value-sense. 
* One nice thing about the term 'gogettum' is that we can speak of the __gogettum's value__, by which we mean the value-referent/wannabe, and can ask whether the gogettum gets the correct value -- without getting all tripped up on too many occurrences of the word 'value', or bogged down with value-referents and value-wannabes.
* Each gogettum has a __name__ that is unique and well-defined. The name appears in code as an identifier. It is also a binary object, which can be passed around, assigned to variables, and so forth.
* Each name is defined (or, we might say, the gogettum is defined) in a __whatsa__, which is also used to define various ancillary objects, such as __dimensions__ (a.k.a. __subscript types__). 
* Whatsas can contain a variety of specifiers, but the whatsa for a gogettum most crucially specifies whether the gogettum is an __input__ to the gogetting or a __calculable__. In the latter case, the whatsa tells the gogetter how to compute the gogettum's value, usually by reference to the names of other gogettums.
* A gogetter, like a Fig Newton, has an __inside__ and an __outside__. The gogetter-system, with its whatsas and whatnot, is the inner component or, simply, the inside. Other client code interacts with the gogetter 'from the outside' -- giving it input values and asking for values back.
* The gogetter itself maintains the state of the gogetting, which it does as lazily as possible. But once it has obtained a gogettum's value, the value is placed in the gogetter's __value cache__ so as to insure its immutability.
* The code construct by which the outer code furnishes the inputs to the gogetting is the __inset__.
* The code construct by which outer code asks the gogetter for the value of a gogettum is a __goget__, or __outer__ or __external goget__. 
* Likewise, the construct by which inner code (in a whatsa) asks for the value of a gogettum is also a __goget__, more specifically an __inner__/__inside__/__internal goget__. 
* You can perform either kind of goget on any gogettum, whether it happens to be an input or a calculated gogettum.
* Circular dependencies among gogettums are an advanced topic. They are easily avoided in most cases, and reliably avoidable in all practical cases I've met so far. But you do occasionally have to keep in mind that they are a danger.
        
So: What is all of this, collectively?

I think of it mostly as a 'framework', for lack of a better term. It's built around the concepts of the gogettum and (especially) the goget. Everything else more or less follows from those.

It gives rise to a programming paradigm, with a quite different take on how to think about code from the other paradigms out there. (Somewhat strangely, it feels more like functional programming than any other paradigm; it seems to have some deep affinities to FP that I don't feel that I have fully understood yet.)

It probably seems underwhelming to those who first read about it! Apart from the jargon, it's all pretty simple and obvious. My suspicion is that this is a strength, in fact, but of course time will tell. It has been my experience that it is hard to get people intrigued by it, but that once they start to use it, they tend to become enthusiastic about it.

The key construct is the goget. It differs from the assignment of a value from one variable to another in that the right-hand side doesn't have to have the value handy going into its execution. In that respect, it more resembles an assignment with a function call on the right-hand side. Its differences from a function call (which is of course how it's implemented) are more subtle. The _name_ plays a mysterious role in the whole business. In some ways, because the name can itself be a run-time, binary object -- a value, in fact -- the goget feels more like calling a function that is stored in a variable of some callable type.

But why should _you_ read about this? A fair question, which I will take up in Part 3.

[This post revised and expanded 12 Sep 2020]