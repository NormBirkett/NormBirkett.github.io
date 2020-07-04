---
title: "Introducing Gogetter, Part 2: A Quick Sketch of How It Works"
date: 2020-07-04
categories: gogetter 
comments: true
---

Here's another way to code the calculation of our finance charge, along with other parts of our credit-card statement--written in an idealized gogetter syntax that is only pseudocode at this point in time. We're going to base this off of our most-explicit-about-the-inputs version of our finance charge calculation from Part 1.

```C++

        gogetter statement () {

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

Okay, now that we've told the system how to calculate all our values, how do we get it to do something with this information? The following pseudocode loads up the calculation's inputs and then retrieves the finance charge and other value-wannabes. (In the real world you would normally find these two actions in separate places in the code.)

```C++

                gogetter stmt(statement);                                               // 1

                stmt.hereis(date_range) << some_statement_date_range;                   // 2
                stmt.hereis(interest_rate_source) << some_interest_rate_source_object;
                stmt.hereis(starting_balance) << some_starting_balance;

                for (size_t i = 0; i < some_collection_of_charges.size(); ++i) {
                        stmt.hereis(charges[i]) << some_collection_of_charges[i];       // 3
                }
                stmt.close(charges);                                                    // 4

                // ... and so for the other collections of transactions

                int_rate_t interest_rate <<= stmt.goget(interest_rate);                 // 5

                auto fin_chg_v <<= stmt.goget(finance_charge);                          // 6

                // Here are some other ways of retrieving the finance charge:
                money_amt_t fin_chg_v_again <<= stmt.goget(finance_charge);             // 7
                const money_amt_t & fin_chg_ref &= stmt.goget(finance_charge);          // 8
                const money_amt_t * fin_chg_ptr *= stmt.goget(finance_charge);          // 9

                money_amt_t fin_chg_assign_to;
                fin_chg_assign_to <<= stmt.goget(finance_charge);                       // 10

                const money_amt_t * fin_chg_ptr_assign_to{nullptr};
                fin_chg_ptr_assign_to *= stmt.goget(finance_charge);                    // 11

                money_amt_t avg_daily_bal <<= stmt.goget(average_daily_balance);        // 12
```

This is also pseudocode. How it translates into C++ syntax is a topic for another day. [And a work in progress, as of this writing.]

Explanations, keyed to the numbered lines in the latter code snippet:

1. We declare a gogetter called 'stmt' that uses the 'statement' definition seen earlier. But what IS this gogetter thing? It is two things. First, it is a system of interdefined value-senses--see the collection of whatsas above.Second, it is the entity that manages that system at run time, and most usefully, goes and gets value-referents (at any rate, value-wannabes) when asked to.

2. The 'hereis' construct gives the gogetter the value of an input. The name of the input is given in the parentheses, and the '<<' is meant to suggest movement of the value (on the right; it can be a literal or, as here, a variable of some kind) into the named value-slot.

3. The name 'charges[i]' has one dimension. That is, each specific value-thing that will get the name 'charges' will be distinguished by one subscript.

4. When 'charges' was defined (in a 'whatsa', earlier), it was given the dimension 'charges_index', which is defined as 'const'. That means the gogetter needs to be told when all the 'charges[i]' values have been supplied, so it can close the dimension and keep it constant for the duration of this gogetting (which is what we call one instance of our grand calculation).

5. Now we switch to retrieving values from the gogetter. The construct by which this is achieved is called a 'goget'. In this case, we get the value of the gogettum that is named within the parentheses.

6. The goget expression has the type of its gogettum (specified earlier in the 'whatsa'), and can therefore serve to feed its value into a variable declared 'auto'.

7. These illustrate the three forms of the goget, all applied to the same gogettum. 7 shows retrieval by value.
8. 8 illustrates getting a reference to the value inside the gogetter's collection of values. 
9. 9 obtains a pointer (to const) instead.

10. A goget can be used as the RHS of an assignment, not just as an initializer.
11. So too can the pointer goget (but not the reference goget, for obvious reasons).

12. Something needs to be said at this point about what actually happens when a goget is executed. Here are the steps:

  * The gogetter looks at the name of the gogettum.
  * It checks whether it has already computed that gogettum.
  * If it has, it just returns the value that is already there in its store.
  * If not, it checks that it has a whatsa for the gogettum.
  * If it doesn't, it signals an error. (How it does this is a question we will defer.)
  * If it does, but the whatsa indicates it's an input gogettum, it signals an error (ditto).
  * If we've reached this point, the gogetter sets about computing the gogettum.
  * This it does (obviously) by executing the instructions in the whatsa.
  * Note that a whatsa can itself contain gogets--on other gogettums.
  * These are executed in a logically recursive manner. (Implementation is another matter, but we will defer discussion of that.)

Out of all of this there emerges an ontology, if you will. Let's summarize it.

  A computation--that is, an instance or occurrence or execution of a computation--is called a _*gogetting*_. The (client) code that specifies all the values to be computed in a particular kind of gogetting is called a _*gogetter system*_ or just _*gogetter*_. Another use of _*gogetter*_ is for the run-time component of the library, which performs all the operations involved in a gogetting by referring to the gogetter system.

  Each value that we might want to compute in a particular computation is represented as a _*gogettum*_, of which the plural is _*gogettums*_. The gogettum is an abstraction of a value, and like 'value', the term is used for value-senses, value-referents, and value-wannabes. One nice thing about using the term 'gogettum' is that we can speak of the gogettum's value, by which we mean the value-wannabe, and can ask whether the gogettum has the correct value--without getting all tripped up on too many occurrences of the word 'value'.

  Each gogettum has a _*name*_ that is unique and well-defined. The name appears in code as an identifier. It is also a binary object, which can be passed around, assigned to variables, and so forth.

  Each name is defined (or, we might say, the gogettum is defined) in a _*whatsa*_, which is also used to define various ancillary objects, such as dimensions. Whatsas can contain a variety of specifiers, but the whatsa for a gogettum most crucially specifies whether the gogettum is an input to the gogetting or a calculated gogettum. In the latter case, the whatsa tells the gogetter how to compute the gogettum's value, usually by reference to the values of other gogettums.

  It is useful to think of the whatsas as constituting the inner code of the gogetter system, whereas other client code interacts with the gogetter 'from the outside'--giving it input values and asking for values back.

  The gogetter itself maintains the state of the gogetting, which it does as lazily as possible. But once it has obtained a gogettum's value, the value is placed in the gogetter's _*value store*_ so as to insure its immutability.

  The code construct by which the outer code furnishes the inputs to the gogetting is the _*hereis*_.

  The code construct by which outer code asks the gogetter for the value of a gogettum is a _*goget*_. Likewise, the construct by which inner code (in a whatsa) asks for the value of a different gogettum is also a goget. When it is necessary to distinguish these, we speak of _*inner*_ or _*internal gogets*_ and _*outer*_ or _*external gogets*_. It probably goes without saying, but I'll say it anyway: You can perform either kind of goget on any gogettum, whether it happens to be an input or a calculated gogettum.
        
So: What is all of this, collectively?

I think of it mostly as a 'framework', for lack of a better term. It's built around the concepts of the gogettum and (especially) the goget. Everything else more or less follows from those.

It also gives rise to a programming paradigm, with a quite different take on how to think about code from the other paradigms out there. (Somewhat strangely, it feels more like functional programming that any other paradigm; it seems to have some deep affinities to FP that I don't feel that I fully understand.)

It probably seems underwhelming to those who first read about it! Apart from the jargon, it's all pretty simple and obvious. My suspicion is that this is a strength, in fact, but of course time will tell. It has been my experience that it is hard to get people intrigued by it. I'll return to that later.

The key construct is the goget. It differs from the assignment of a value from one variable to another in that the right-hand side doesn't have to have the value handy going into its execution. In that respect, it more resembles an assignment with a function call on the right-hand side. Its differences from a function call (which is of course how it's implemented) are more subtle. The _name_ plays a mysterious role in the whole business. In some ways, because the name can itself be a run-time, binary object--a value, in fact--the goget feels more like calling a function that is stored in a variable of some callable type.

But why should *you* read about this? A fair question, which I will take up in Part 3.
