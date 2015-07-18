---
layout: post
title: A Formal Programming Language
active: projects
categories:
 - Language
tags:
 - compiler
 - concurrency
 - csp
 - hoare
---

####The Language
First, an example. No syntax highlighting yet! This is the as of yet impractical Hello world. There are no strings or arrays yet, just signed and unsigned system types, and booleans. The variables in this example don't do anything except to demonstrate the declaration of variables. Comments are Ada-style. The language is imperative and strongly typed, but has type inference for integer literals. 

<div class="highlight">
<pre>
<code>
process Main (Stdin?: u8, Stdout!: u8) is
begin
	let x: u8;
	let y: u32;
	-- Hello, world!
	Stdout!72 -> Stdout!101 -> Stdout!108 -> Stdout!108 ->
	Stdout!111 -> Stdout!44 -> Stdout!32 -> Stdout!119 -> 
	Stdout!111 -> Stdout!114 -> Stdout!108 -> Stdout!100 ->
	Stdout!33 -> Stdout!10 -> SKIP
end Main
</code>
</pre>
</div>

This language draws from [Hoare's CSP](https://en.wikipedia.org/wiki/Communicating_sequential_processes), Ada, and Rust. Originally I was using a syntax more like CSP<sub>M</sub>, but after reading [Alain Martin's 2012 paper](http://www.async.caltech.edu/Pubs/PDF/chpasync2012.pdf) I chose a syntax more like his. I feel that it provides a more object-oriented feel, and should enhance code reuse. 

CSP programs are composed of processes, which are program parts that communicate through message passing. Processes can instantiate other processes as children, which all run in parallel. In this dialect, channels are the means of communication between processes. Channels are defined by connecting the ports of children processes or those of the parent process. 


####Another Example

<div class="highlight">
<pre>
<code>
process OtherProc (L?: u8, R!: u8) is
begin
	let x: u8;
	L?x -> R!x -> SKIP
end OtherProc

process Main (Stdin?: u8, Stdout!: u8) is
begin
	let x: u8;
	instance p1: OtherProc;
	instance p2: OtherProc;
	connect Stdin, p1.L;
	connect p1.R, p2.L;
	connect p2.R, Stdout;
	SKIP
end Main
</code>
</pre>
</div>

The prefix (```->```) operation is how a process engages in events, such as sending and receiving messages on channels. This program accepts one character from standard input and prints it to the console in a very indirect way, to demonstrate instances and connections. The question mark signifies receiving a value into a variable and the exclamation mark signifies sending the result of an expression on a channel. There are also loops, both unconditional and conditional. Unconditional loops are written ```*[[ <process_expression> ]]```, where a process expression is a combination of operations such as prefix, loops, and guarded expressions. A conditional loop is written in the form ```*[<guard> -> <process_expression>]```, where the guard is a boolean expression. 

####Why

Software bugs are a huge problem, especially in medical devices, and devices in similar life-sustaining roles. It is unacceptable for a medical device to harm the patient. Languages used in such fields should play a role in the safety of the end user. They should't allow undefined behavior the way C does. Such languages should be amenable to analysis and verification so that correct functionality and safe execution can be ensured. 

####Current Status And Next Steps

Right now, the language is running on a very simple stack VM. There is _very_ basic threading. My current focus is on native code generation, first for x86-64, and then an embedded platform (I just found my Duemilanove, so I'm thinking ATmega168). I'm bootstrapping the code generation with NASM, by piping out the assembly over stdout. Ultimately I want to add assertions and proofs to the language, allowing for formal verification of programs. 

I haven't released the source or an executable yet, but stay tuned!

