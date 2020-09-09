---
title: "Intro to Gogetter, Part 1: Values Clarification"
layout: post
date: 2020-09-05
categories: gogetter values
comments: true
---

### An example, and how we might code it at present

We need an example to work with here.  So:  Suppose we're writing code to calculate data to go on a credit-card statement. Consider the 'finance charge'. How is this calculated?  Well, the code will obtain several other pieces of data, such as
* the average daily balance on the account
* the interest rate on the account
* the number of days in the statement period

and it will do some calcualtion on them that results in the finance charge.

For the last step, we might write something like
<!--more-->

```c++
money_t finance_charge(money_t avg_daily_bal, int_rate_t interest_rate, int days_in_period) {
  return avg_daily_bal * interest_rate * days_in_period / 365.0;
}
```

That function would be called by some client code that might look like this:

```c++
// Calculate all the stuff, Version 1
money_t average_daily_balance{ avg_daily_balance(stmt_date_range) };
int_rate_t interest_rate{ interest_rate_during(stmt_date_range) };
int days_in_stmt_period{ stmt_date_range.end() - stmt_date_range.begin() };
money_t fin_charge{ finance_charge(average_daily_balance, interest_rate, days_in_stmt_period) };
```

This code will provoke questions.

* Where is the avg_daily_balance function getting the balances?

  Let's say all of this is inside some clas, say cc_statement_t, and so avg_daily_balance has access to the statement's member variables, directly or indirectly.

* But maybe we don't like the resulting code. (I don't.) Couldn't we reorganize things to make each calculation more explicit about what its inputs are? 

  Yes, we could, and that would be better. We will do just that in a moment.

* Why do you bother to declare all these variables for average daily balance, interest rate, and so on?

  Good question. We obviously don't have to. But it has certain advantages. First, the intermediate results we need en route to the finance charge are clearly named in the code. That makes the semantics of the calculation unmistakable to the reader. Second, and more important, every one of these intermediate results is also going to appear on the statement. So it makes sense to grab each one and then hold onto it, so it's available for however many times we need it in our calculations, and can eventually be returned somehow as part of the complete statement data set.

But let's try eliminating most of the variables, just to see what we get.

```c++
// Calculate all the stuff, Version 2
money_t fin_charge { 
    finance_charge(avg_daily_balance(stmt_date_range), 
            interest_rate_during(stmt_date_range), 
            stmt_date_range.end() - stmt_date_range.begin()) 
};
```

Dig the efficiency! Just one big initialization! True, though personally I think it's not very readable. Even more seriously, once we get to the place where we want to capture, say, the average daily balance in order to put that on our credit-card statement, we will have to invoke 

```c++
avg_daily_balance(stmt_date_range)
```

again. Whenever you invoke a function again, you introduce the possibility that it will do something slightly different. Which means the statement might show a different interest rate from the one that was actually used in the calculation of the finance charge. The DRY Principle (my version: 'Don't repeat yourself, don't repeat yourself, don't repeat yourself!') is usually thought of in relation to _writing_ the code, but it is only slightly less important when _executing_ the code. __Repeating yourself is an opportunity to fail to repeat yourself. So don't repeat yourself, and you cannot fail to repeat yourself.__ Deep, man.

Let's take our code in a different direction.

```c++
// Calculate all the stuff, Version 3
const money_t average_daily_balance{ avg_daily_balance(stmt_date_range) };
const int_rate_t interest_rate{ interest_rate_during(stmt_date_range) };
const int days_in_stmt_period{ stmt_date_range.end() - stmt_date_range.begin() };
const money_t fin_charge{ finance_charge(average_daily_balance, interest_rate, days_in_stmt_period) };
```

This is a step in the right direction! We have captured each value that we need just once, and have made it obvious to the reader of the code that each of the resulting values is immutable. It is now easy to prove that the value of the interest rate that was used in calculating the finance charge is in fact the same value that gets sent off separately to the statement to be formatted and presented to the cardholder. That's a big advantage!

But let's go a bit further here and make the ultimate data dependencies fully explicit.

```c++
// Calculate all the stuff, Version 4
const money_t average_daily_balance{ 
    avg_daily_balance(starting_balance, charges, cash_advances, payments, credits, stmt_date_range) 
};
const int_rate_t interest_rate{ interest_rate_source.interest_rate_during(stmt_date_range) };
const int days_in_stmt_period{ stmt_date_range.end() - stmt_date_range.begin() };
const money_t fin_charge{ finance_charge(average_daily_balance, interest_rate, days_in_stmt_period) };
```

Now we've made the data relationships within our code context clearer. Questions will remain, however. What is this magical 'interest_rate_source' object? Fair question. But at least it has a tidy abstraction around it: We have encapsulated the job of using whatever data is needed to calculate interest rates on credit card accounts for specified date ranges. And the rest of this code is pretty clear.

### Thinking about the values

How could code like this be made _radically_ better? In other words, if we didn't just want to continue to make minor readability tweaks, but really shake things up, what might we do?

Let's reflect on the snippets we've seen. What's central to them? _The values we want to calculate._ 

What can we say about these values? Well, first, we can describe them in ways that enable us to understand them. Once I know that I have something called an 'interest rate' and something called 'days in statement period' and so forth, then if I have some knowledge of the problem domain, I will know what to do with these values to produce yet other specific values--of which I have similar descriptions that enable me to understand them.

But this business of _understanding values_ needs more elucidation. To get there, let's move from the code perspective to the run-time perspective. Each time we run our calculation from a specific set of inputs, we notice that the values have two 'manifestations', we might say. One level is their _meaning_. This is given by the _description_ that we referred to in the paragraph before this one -- that is, various words and phrases in our coding metalanguage (here business English) -- and by names of variables, classes, objects, and functions in our code. The second manifestation is awkward to talk about, because it is the specific number (or value) that gets calculated, and speaking of 'the value of a given value' sounds clumsy. But each time we run our calculation, we get some number with associated units etc. and think of that as 'the value' or 'the result'. That is, of the value we were looking for. Awkwardly.

To clarify this terminology, let's borrow some jargon from the philosophy of language and call the meaning manifestation the 'sense' of the value, or __value-sense__, and the ideal/correct resulting-number-or-other-datum-plus-units manifestation the 'referent' of the value, or __value-referent__. So our 'meaning of the value' is its value-sense, and the correct 'value of the value' is its value-referent. And let's further augment this lingo with some jargon _not_ from the philosophy of language: We will refer to the value actually calculated by a program as the __value-wannabe__. 

The picture then looks like this:

* Knowing a _value-sense_ gives us understanding of the value.
* This understanding of the value tells us how to calculate the _value-referent_ from specific inputs. 
* It also therefore helps us code its calculation.
* And then when we run the code on specific inputs, we get a _value-wannabe_, and our understanding of the value enables us to check the value-wannabe for correctness -- because, as we saw just above, it tells us how to calculate the _value-referent_.

Note that given a set of specific inputs, the value-referent is immutable. It sort of floats out there in some kind of ideal Platonic realm being eternally correct for those inputs. Or to revert to our example, given the inputs, there is one and only one correct finance charge. Let's suppose that on one occasion, it's $42.00, or forty-two US dollars. That is true before we have gotten around to calculating it. And it remains true even if we calculate it wrong.

The eternal immutability of each such value-referent is easy to lose sight of, because we start the calculation not knowing it, and we come to find out what it is by performing the calculation in time. In fact, in olden times, when memory was much more scarce and expensive, we all learned to designate some variable, put some starting value in it, and then write code that did things to that variable until at some point in time it came to contain the correct finance charge. In other words, we tended to do things like this:

```c++

// The way we did it in the bad old days
double x = 0.0;
x = avg_daily_balance(stmt_date_range);
write_to_statement("Avg daily balance: ", x);
x *= interest_rate_during(stmt_date_range);
write_to_statement("Interest rate: ", x);
x *= ((double) stmt_date_range.end() - stmt_date_range.begin()) / 365.0;
write_to_statement("Finance charge: ", x);
```

So _x_ passed through several states on its way to containing the finance charge value-wannabe. Which is one reason it had a lousy name like 'x'!

What was really going on in code like that was not the computation of the one value, but the computation of several intermediate values that all happened to occupy the same piece of memory for short periods. But we tended to think of it as something vaguely akin to the value emerging gradually, in much the way that an orange growing on a tree ripens until it is ready to be eaten. With the benefit of hindsight, we now have good reasons to conclude that we were not thinking about this in the most useful way. So we don't think that way any more (one hopes).

Now we've gotten away from recycling memory in that fashion, but I'm not sure we've ever gotten around to thinking entirely clearly about values. I certainly don't see a lot of clarity out there on these subjects--with a few notable exceptions.

So this business of (meaningful) value-senses and (initially unknown but immutable) value-referents is one thing we can learn from our simple example. Another is that what tends to clarify value-calculation code like this is seeing the functional (in the math/logic sense) relationships among the value-senses. That is, which values are inputs to the calculations of what other values? In our example:

* _The daily balances_ are a function of _the starting balance_, _the transactions_, and _the statement-date range_.
* _The average daily balance_ is a function of _the daily balances_.
* _The interest rate_ is a function of _the terms of the account_ and _the statement-date range_.
* _The days in statement period_ is a function of the _statement-date range_.
* _The finance charge_ is a function of _the average daily balance_, _the interest rate_ and _the days in statement period_.

When we understand that, we understand much more clearly the calculation we need to code.

### Coding with the values

It was thinking along these lines that led me some years ago to wonder, Wouldn't it be nice if we could code in 'values'? Not just in the limited sense of giving names to variables and functions and whatnot, but in a stronger sense: Giving names to each value in our calculation, and then telling the system how to compute it that value in terms of other such named values.

I asked several paragraphs ago if we could find a radically better way of coding value calculations. The 'wouldn't it be nice' of the previous paragraph is how I propose to get there. As you have perhaps guessed, this is where Gogetter comes in.

Stay tuned...

