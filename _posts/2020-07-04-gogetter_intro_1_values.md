---
title: "Introducing Gogetter, Part 1: Values Clarification"
date: 2020-07-04
categories: gogetter values
comments: true
---

We'll start with an example that we can explore thoroughly in the course of this post.

Suppose we're writing code to calculate data for a credit-card statement. Consider the 'finance charge', otherwise known as interest. How is this calculated? Well, the code rounds up several other pieces of data, such as
* The average daily balance on the account
* The interest rate on the account
* The number of days in the statement period
* The interest-calculation terms applicable to the account
and does something with them so that the finance charge pops out. The last item on the list is often just assumed in a given piece of code, so we'll ignore it for now.

To program this calculation, we would typically write something like this.

```C++

money_t finance_charge(money_t avg_daily_bal, int_rate_t interest_rate, int days_in_period) {
   return avg_daily_bal * interest_rate * days_in_period / 365.0;
}
```

And that of course would be called by some client code that might look something like this:

```C++

// The 'calculate all the stuff' code up to the finance charge calculation, Version 1
money_t average_daily_balance{ avg_daily_balance(stmt_date_range) };
int_rate_t interest_rate{ interest_rate_during(stmt_date_range) };
int days_in_stmt_period{ stmt_date_range.end() - stmt_date_range.begin() };
money_t fin_charge{ finance_charge(average_daily_balance, interest_rate, days_in_stmt_period) };
```

Programmers are not shy about questioning the way [other people's] code is written, so this snippet is likely to be met with questions along these lines:

* Where is the avg_daily_balance function getting the balances?

  Let's say all of this is inside a class cc_statement_t, and so avg_daily_balance has access to the statement's member variables, directly or indirectly.

* But maybe we don't like the resulting code. (I don't.) So couldn't we reorganize things to make each calculation more explicit about what its inputs are? 

  Yes, we could, and that would be better. In fact, we will do that in a moment.

* Why do you bother to declare all these variables for average daily balance, interest rate, and so on.

  Good question. We obviously don't have to do that. But it has at least these two advantages. First, it means that the intermediate results we need en route to the finance charge are clearly named in the code. That makes the semantics of the calculation unmistakable to the reader. Second, and more important, every one of these intermediate results is also going to appear on the statement. So we would like to grab each one and save it as we need it, so it's available for however many times we need it in our calculations, and can eventually be returned somehow as part of the complete statement data set.

Lines of questioning like these suggest a couple of other ways we could have written our outer (client) snippet.

```C++

// The 'calculate all the stuff' code up to the finance charge calculation, Version 2
money_t fin_charge{ finance_charge(avg_daily_balance(stmt_date_range), interest_rate_during(stmt_date_range), 
  stmt_date_range.end() - stmt_date_range.begin()) };
```

Dig the efficiency! Fewer lines of code! True, though personally I think it's well on its way to being unreadable. (So much for code metrics!) More seriously, once we get to the place where we want to capture, say, the average daily balance in order to return that to our client, we will have to invoke 

```C++

avg_daily_balance(stmt_date_range)
```

again. And whenever you invoke a function again, you introduce the possibility that it will do something slightly different. Which means the statement might show a different interest rate from the one that was actually used in the calculation of the finance charge. The DRY Principle (my version: 'Don't repeat yourself, don't repeat yourself, don't repeat yourself!') is usually thought of in relation to writing the code, but it is only slightly less important when executing the code. Repeating yourself is an opportunity to fail to repeat yourself. So don't repeat yourself, and you cannot fail to repeat yourself.

Hmmm. Let's take our calculating-all-the-stuff code in a different direction.

```C++

// The 'calculate all the stuff' code up to the finance charge calculation, Version 3
const money_t average_daily_balance{ avg_daily_balance(stmt_date_range) };
const int_rate_t interest_rate{ interest_rate_during(stmt_date_range) };
const int days_in_stmt_period{ stmt_date_range.end() - stmt_date_range.begin() };
const money_t fin_charge{ finance_charge(average_daily_balance, interest_rate, days_in_stmt_period) };
```

In my opinion, this is a step in the right direction. We have captured each value we are going to need just once, and have made it obvious to the reader of the code that each of these values is immutable. It is now very easy to prove that the value of the interest rate that was used in calculating the finance charge was in fact the same value that was sent off separately to the statement to be formatted and presented to the cardholder. And that's a big advantage!

But let's go a bit further here in making our calculation block clear to the reader.

```C++

// The 'calculate all the stuff' code up to the finance charge calculation, Version 4
const money_t average_daily_balance{ 
    avg_daily_balance(starting_balance, charges, cash_advances, payments, credits, stmt_date_range) 
};
const int_rate_t interest_rate{ interest_rate_source.interest_rate_during(stmt_date_range) };
const int days_in_stmt_period{ stmt_date_range.end() - stmt_date_range.begin() };
const money_t fin_charge{ finance_charge(average_daily_balance, interest_rate, days_in_stmt_period) };
```

Now we've made the data relationships within our cc_statement_t clearer. Questions will remain, however. What is this magical 'interest_rate_source' object? Fair question. But at least it has a tidy abstraction around it: We have encapsulated the job of using whatever data is needed to calculate interest rates on credit card accounts for specified date ranges. And the rest of this code is pretty clear. (There are interesting principles of programming that are illustrated here, but we need to move on, alas.)

How could code like this be made RADICALLY better? In other words, if we didn't just want to continue to make minor readability tweaks, but really shake things up, what might we do?

Let's reflect on the snippets we just looked at. 

What's central to them? The values we want to calculate. These values have descriptions that enable us to understand what they are. And each time we perform our calculation, the code (we hope) calculates each of them correctly.

Within a given calculation--that is, within a given execution of our calculation from a specific set of inputs--the values have two 'levels', we might say. One level is their meaning. This is given by various words and phrases in our coding metalanguage (here business English), and by names of variables, classes, objects, and functions in our code. The other level is awkward to talk about, because it is the value that gets calculated, and speaking of 'the value of a given value' sounds weird! But each time we run our calculation, we get some number with associated units etc. and think of that as 'the value'. That is, of the value we were looking for. Awkwardly.

Let's borrow some jargon here from philosophy of language and call the meaning level the 'sense' of the value, and the number-or-other-data-plus-units level the 'reference' of the value, while the number-... thing is the 'referent'. From here on, I will speak of the one way of talking about the value as the value-sense, and the other way, the ideal result, as the value-referent.

And we'll augment this with some jargon NOT from the philosophy of language: We will refer to the value actually calculated by a program as the value-wannabe. 

The picture is that the sense of each value is its meaning, in the sense (!) that it enables us to figure out how to code the calculation of the referent, and how to figure out if we got it right. And each time we run the calculation, there is a single correct value-referent for each of our value-senses. But the value that is actually calculated by our calculation we call the value-wannabe. Whether our code is correct, then, boils down to the question of whether our value-wannabes are equal to the corresponding value-referents.

The value-referent for each calculation-execution is immutable, note, as it just floats out there in some kind of Platonic realm of ideals being eternally correct. That is, given the inputs, there is a correct finance charge. Let's suppose that on one occasion, it's (surprise, surprise!) $42.00, or forty-two U.S. dollars. That is true before we have gotten around to calculating it. And it remains true even if we calculate it wrong.

The immutability of each such value-referent is easy to loose sight of, because we start the calculation not knowing it, and we come to find out what it is by performing the calculation. When memory was much more scarce and expensive, we all learned to designate some variable, put some starting value in it, and then write code that does things to that variable until at some point in time it starts to contain the correct finance charge. In other words, we tended to do things like this:

```C++

// The way we did it in the bad old days
double finchg = 0.0;
finchg = avg_daily_balance(stmt_date_range);
finchg *= interest_rate_during(stmt_date_range);
finchg *= ((double) stmt_date_range.end() - stmt_date_range.begin()) / 365.0;
```

So 'finchg' passed through several states on its way to containing the finance charge value-wannabe.

What was really going on in code like that was not the computation of the one value, but the computation of several intermediate values that all happened to occupy the same piece of memory for short periods. But we tended to think of it as something vaguely akin to the value changing, in much the way that an orange growing on a tree ripens until it is ready to be eaten. There are lots of good reasons to conclude that we were not thinking about this in the most clarifying way!

Now we've gotten away from recycling memory in that fashion, but I'm not sure we've ever gotten around to thinking entirely clearly about values. I certainly don't see a lot of clarity out there on these subjects--with a few notable exceptions.

So this business of (meaningful) value-senses and (initially unknown but immutable) value-referents is one thing we can learn from our simple example. Another is that what tends to clarify value-calculation code like this is seeing the functional (in the math/logic sense) relationships among the value-senses. That is, which values are inputs to the calculations of what other values? When we understand that, we understand the calculation we need to code.

It has been thinking along these lines that led me some years ago to think, Wouldn't it be nice if we could code in 'values'? Not just in the limited sense (!) of giving names to variables and functions and whatnot, but in a stronger sense: Giving names to each value in our calculation, and then telling the system how to compute it in terms of other such named values.

I asked several paragraphs ago if we could find a radically better way of coding value calculations. The 'wouldn't it be nice' of the previous paragraph is how I propose to get there. And as you have perhaps guessed, this is where Gogetter comes in.

Stay tuned...

