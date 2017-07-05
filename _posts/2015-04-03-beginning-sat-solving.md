---
layout: post
title: Beginning SAT-Solving
categories:
 - Formal methods
tags:
 - formal
 - python
 - sat solving
 - logic
---

Recently Microsoft open-sourced [Z3](https://github.com/Z3Prover/z3/), an SMT-solver. SMT-solving is a formal method used in proving theorems and code, and is an extension of SAT-solving. A few days later, Microsoft also released the [Lean Prover](https://leanprover.github.io/), a different kind of theorem prover. This gave me the push to learn a bit more about SAT solving. As I come to terms with this method myself, I thought I would share my experience here. 

#### Introduction

Before you get started, you can get the SAT-solving tool we're using [here](https://github.com/ueliem/pysat/tree/a8a2b7463eb9b97866c9f713df5afc6ba1d1a230)!

Boolean Satisifiability (SAT) solving is a a formal method that has application in verifying software function. However, the theory has been around for long before. [The Boolean Satisfiability Problem](http://en.wikipedia.org/wiki/Boolean_satisfiability_problem) is a question whether a given Boolean equation can have its variables replaced with some combination of ```True``` and ```False``` such that the equation _can_ be evaluated to ```True```. 

For the purposes of this post, I wrote a [very simple SAT-solver](https://github.com/ueliem/pysat/tree/a8a2b7463eb9b97866c9f713df5afc6ba1d1a230). It's ugly, but it _works_ and is accessible to anyone familiar with Python. It uses brute force to determine whether any values satisfy the boolean equation supplied. For our purposes, this is fine, because we are using small equations. As you can guess, the runtime would be worse for equations with very large numbers of inputs. Real SAT-solvers use better algorithms or heuristics to solve more efficiently. I chose to keep the code simple for comprehension. The solver also cannot handle parentheses, so don't use them! Like I said, lazy. Here is a bit of theory on how this all works. 

#### Theory

Satisfiability solving is based on [propositional logic](http://en.wikipedia.org/wiki/Propositional_calculus). If you are familiar with Boolean algebra you already know the basics. Propositional logic is a decision process based on True or False values. 

The following are the basic operators of the propositional logic language and their Python equivalents, where $a$ and $b$ are variables:

1. $$a \land b$$, corresponding to ```a and b```

2. $$a \lor b$$, corresponding to ```a or b```

3. $$\sim a$$, corresponding to ```not a```

There are other operations used but I will not be covering them in this post. They are more complex, compounds of these basic operations anyway. 

These basic operations can be arranged into equations such as ```a and b or not c```. The values ```True``` and ```False``` can be plugged in to the equation and the equation can be evaluated to either ```True``` or ```False```. Satisfiability is the property of the equation whether the equation can be evaluated to ```True```. 

What makes the "Boolean satisfiability problem" a "problem" is that it is possible to write a Boolean equation that never evaluates to ```True```. A very simple example would be:

{% highlight python %}
a and not a
{% endhighlight %}

Manually plug in ```True``` for ```a```, and then try ```False```. Both evaluate to ```False```. This means the equation is not satisfiable. If you were anticipating a ```True``` to ever be the result, you would be out of luck. 

#### Using The Solver

If you haven't already, you can get the solver from [here](https://github.com/ueliem/pysat/tree/a8a2b7463eb9b97866c9f713df5afc6ba1d1a230).

The solver is a Python program run from the shell. The first argument to the program is the Boolean equation you want to be satisfied. For example:

{% highlight bash %}
python pysat.py "a and b"
{% endhighlight %}

#### Proving Code?

Getting to the point of this post: let's make sure code isn't broken. SAT solving can only handle booleans; to handle numbers, they have to be translated down to binary and then each bit can be treated as a boolean value. That's a bit (ha) complicated for this post. Let's do something simple. 

{% highlight python %}

def myfunc(x, y):
	if x and y:
		return True
	elif x and not x:
		return False
	else:
		return False

{% endhighlight %}

There's something wrong with this code. The ```elif``` block will never return ```False```. Assuming the contents of that block were application logic or other important code, there would be a big problem. 

Let's try it out on the SAT-solver and see if this is satisfiable. 

{% highlight bash %}
python pysat.py "x and not x"
{% endhighlight %}

You should see the following:
{% highlight bash %}
Evaluating "x and not x"...
The equation is not satisfiable.
{% endhighlight %}

Turns out, it's not satisfiable. Simply, x cannot be both ```True``` and ```False``` at the same time. ```True and not True``` simplifies to ```True and False```, which is ```False```, and ```False and not False``` simplifies to ```False and True```, which is also ```False```. Knowing the behavior of an ```if``` statement, which only executes the code inside when the condition evaluates to ```True```, it is clear that because this boolean equation is not satisfiable, the code within it is unreachable. However, the code is not seen as troublesome by the interpreter (or, say, compiler)! In this case someone may have just made a single typo, meaning to put ```y``` instead of ```x``` in one spot. It's possible that in a more complicated scenario that someone made an error in their algorithm or logic and didn't know they made a mistake. A SAT-solver or related tool (in the category of static analyzer) can solve headaches like this. 

#### Conclusion

While this post presents a trivial case, I think that it shows the merits of SAT-solving in verifying code. This method shows a way to catch certain kinds of logic bugs, but has a short coming in that it is compicated to verify anything involving numbers and comparisons. Obviously, automated analysis of code would be better than doing it manually like we did in this post. Even better would be integrating this right into the front end of the interpreter/compiler. 

In a future post I hope to address the short comings of SAT-solving by exploring SMT-solving, a method designed to do just that. 

{% include mathjax.html %}
