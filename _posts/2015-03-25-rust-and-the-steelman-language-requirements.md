---
layout: post
title: Rust and the Steelman Language Requirements
categories:
 - Rust
tags:
 - rust
 - steelman
 - ada
 - dod
---

####Introduction

Getting deeper into embedded systems, I'm approaching a point where quality assurance is becoming more important. Some of my code may end up in life-sustaining devices (I am a biomedical engineering student). I've been getting to know the [Rust language](http://www.rust-lang.org/) and I have enjoyed using it for desktop programming. A while ago I wrote a bit of a compiler project in it, as well as some less serious projects. The safety features of Rust had me thinking about using it for embedded programming in the context of writing real-time code for medical devices. [I'm not the first person to think of this. ](http://zinc.rs/)

Now, I am new to high assurance situations. I've been searching for documents to read to get familiar with it and I eventually came across the Steelman Language Requirements. The United States Department of Defense has funded a lot of inquiry and development in high assurance programming for aerospace control systems. A lot of overlap exists with medical devices, in that while they are running, they are life-sustaining; if they fail, lives can be lost. The Steelman Language Requirements led to the creation of [Ada](http://en.wikipedia.org/wiki/Ada_%28programming_language%29), a programming language used in these situations. 

The Steelman Language Requirements lists features of a programming language that ensure that the resulting program matches the programmer's intentions and has minimal bugs. I'm not familiar with the ins and outs of Ada, but I was curious if Rust could effectively be used in the same areas as Ada. 

Since the Requirements are actually really long, I didn't read every bit. I chose to write about the parts I did read and felt that Rust did not meet well enough. 

####1H. COMPLETE DEFINITION

> #####[1H. COMPLETE DEFINITION](http://www.iment.com/maida/computer/steel/steelman.htm#s1h.)
> The language shall be completely and unambiguously defined. To the extent that a formal definition assists in achieving the above goals (i.e., all of section l), the language shall be formally defined. 

Currently, the language is not fully formally defined. The Requirements intend that a language should be able to be reimplemented from scratch using only the formal definition of the grammar and language features, while having identical behavior. Rust has been improving in this area gradually, but doesn't yet completely meet this. [The Rust Grammar page](http://doc.rust-lang.org/grammar.html) shows the currently unfinished formal grammar of the language; as you can see pieces are missing. Rust would have to complete this grammar at the least to meet this requirement. Meeting this requirement means that the developer is not locked into one particular compiler; standards ensure that the code will compile predictably across any compiler implementation. 

####2A. CHARACTER SET

<blockquote>
<a href="http://www.iment.com/maida/computer/steel/steelman.htm#s2a."><h5>2A. CHARACTER SET</h5></a>
<p>The full set of character graphics that may be used in source programs shall be given in the language definition. Every source program shall also have a representation that uses only the following 55 character subset of the ASCII graphics: </p>

<img src="http://www.iment.com/maida/computer/steel/images/characterset.jpg" alt="55 character set" />

<p>Each additional graphic (i.e., one in the full set but not in the 55 character set) may be replaced by a sequence of (one or more) characters from the 55 character set without altering the semantics of the program. The replacement sequence shall be specified in the language definition. </p></blockquote>

Any language that uses curly brackets violates this requirement. I could see that perhaps there is an issue in the programmer mismatching brackets and unintentionally placing certain code in the wrong code block. Probably minor, but Rust also uses some other characters like the exclamation mark that are not in the 55 character set. I don't think this is a really big deal. The 55 character set also doesn't include lowercase letters, only uppercase, so take this as you will. 

####2C. SYNTACTIC EXTENSIONS

> #####[2C. SYNTACTIC EXTENSIONS](http://www.iment.com/maida/computer/steel/steelman.htm#s2c.)
> The user shall not be able to modify the source language syntax. In particular the user shall not be able to introduce new precedence rules or to define new syntactic forms. 

So, Rust allows to language extension in the form of macros and compiler plugins. Macros can be useful because they are evaluated at compile time, but they do add complexity and possible faults. Internal to a macro call can be new syntax that is not a part of the Rust standard. The parsing within is written in Rust and can generate code. While it is nice that they are more syntactic and actually a part of the compiler, unlike the C preprocessor, generating code can introduce unclear issues that could be harder to understand and troubleshoot. 

Rust's macro system does have a lot of benefits that I mainly see in the example of ```println!``` used in place of C's ```printf```. Variable argument number is unsafe to use in C, but because ```println!``` is expanded and checked at compile time it is much safer. 

####3-1G. FIXED POINT SCALE & 3-1H. INTEGER AND FIXED POINT OPERATIONS

> #####[3-1G. FIXED POINT SCALE](http://www.iment.com/maida/computer/steel/steelman.htm#s3-1g.)
> The scale or step size (i.e., the minimal representable difference between values) of each fixed point variable must be specified in programs and be determinable during translation. Scales shall not be restricted to powers of two.

> #####[3-1H. INTEGER AND FIXED POINT OPERATIONS](http://www.iment.com/maida/computer/steel/steelman.htm#s3-1h.)
> There shall be integer and fixed point operations for modulo and integer division and for conversion between values with different scales. All built-in and predefined operations for exact arithmetic shall apply between arbitrary scales. Additional operations between arbitrary scales shall be definable within programs. 

Rust lacks any true built-in fixed-point math capability. I mean, as in C, you can pretend that there are fixed-point data types and just treat them nearly the same in math as regular integer types. However, it's hard to use a scale other than powers of two, and if you use any other you have to keep track of which variable has which scale yourself. This is error prone and impractical. 

Fixed-point math is important because my current work is on an ARM Cortex-M, which doesn't have a floating point unit. Doing floating point math on the main CPU is not performant, and generally not a good idea in real time systems. 

####Conclusion

I don't think that Rust meets these requirements well enough. I think Rust is a great language, and I am definitely going to continue to write in it. It's best to say that Rust brings extra security to and reduces the amount of bugs in programs that would be written in C or C++, without sacrificing convenience. The Rust project emphasizes this on their website and documentation as the actual niche for the language. 

This is more of a wishlist thing, but it would be nice to have compiler support for fixed-point math. Rust is still young and there is plenty of time for the language to grow and mature. I'm looking forward to the upcoming 1.0 release!

