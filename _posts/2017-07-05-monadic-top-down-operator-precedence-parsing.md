---
layout: post
title: Monadic Top Down Operator Precedence Parsing
active: projects
categories:
 - Language
tags:
 - compiler
 - monad
 - sml
---

I'm designing a new ML-based language that has user-defined infix operators. 
The prototype is written in SML but I ultimately want the language to be self-hosting. 
Right now I am rewriting the parser, which was originally written with mllex/mlyacc, with a custom parser combinator library. 
Parser combinators seem easier to port than a parser generator (although I'm not sure how true that really is), and they do not need an extra build step to compile the parser from a grammar. 
I implemented a parser combinator library in SML based on [Monadic Parsing in Haskell](http://www.cs.nott.ac.uk/~pszgmh/pearl.pdf) and [Monadic Parser Combinators](http://www.cs.nott.ac.uk/~pszgmh/monparsing.pdf) by Hutton and Meijer. 
There is a lack of information on parsing infix operators with varying precedence and associativity that plays well with other forms of expressions in the parser. 
In a previous Rust-based project I used the [precedence climbing algorithm](https://en.wikipedia.org/wiki/Operator-precedence_parser#Precedence_climbing_method) for the grammar of arithmetic. 
It is a simple and accessible algorithm. 
Since SML is not imperative, I decided to extend the parser combinators into a parser monad. 
SML doesn't have monads built in but they can be added with a simple signature. 

```sml
infixr 1 >>=

signature MONAD =
sig
  type 'a monad

  val return : 'a -> 'a monad

  val >>= : 'a monad * ('a -> 'b monad) -> 'b monad

end
```

This is an excellent use case for monads. 
Monads hide the failure case which means less boilerplate dealing with ```'a option```; you just write code for the successful parse case. 
Sequencing parser monads also represents the list processing in the parser well. 
A sequence of monadic parsers only succeeds if all of them succeed. 
The core code is quite small and neat. 
However, the many nested anonymous functions from binding monads made me wish for Haskell's do-notation, which SML does not have. 

Here is pseudo code for the imperative algorithm. 
RHS/LHS refer to the left and right hand side of an operator. 
An operator's precedence determines how closely the operator binds the RHS/LHS. 
A higher precedence binds more tightly. 
For example, multiplication and division bind more tightly than addition and subtraction in conventional arithmetic and most programming languages. 
For our purposes, these main operators will have left [associativity](https://en.wikipedia.org/wiki/Operator_associativity) only. 

```
function prec_parse (level : int) (input : char stream) : AST.expr

	LHS = Parse an atom from input

	While next token is operator with precedence >= current level, 
		op = Parse an operator from input
		op_prec = precedence (op)
		RHS = prec_parse (if op associates to the left then op_prec + 1 else op_prec) input
		LHS = make_node (op, LHS, RHS)

	return LHS
```

This is the main fragment of the SML implementation. Below it is an explanation of the conversion from the imperative algorithm to the monadic one. 

```sml
...

and binop level input = (
let 
  val opr = [
  ("+", (1, LeftAssoc, AST.Plus)),
  ("-", (1, LeftAssoc, AST.Sub)),
  ("*", (2, LeftAssoc, AST.Mult)),
  ("/", (2, LeftAssoc, AST.Div))
  ]

  fun retrieve x = 
	case List.find (fn y => #1 y = x) opr of
	  SOME z => if #1 (#2 z) >= level then SOME (#2 z) else NONE
	| NONE => NONE
in
  token sigil >>= (fn x =>
  (case retrieve x of
	SOME y => result y
  | NONE => fail))
end
) input

and binary input = prec_parse minprecedence input

and prec_helper level input = (
  binop level >>= (fn (p,a,b) =>
  prec_parse (case a of LeftAssoc => (p + 1) | _ => p) >>= (fn rhs =>
  result (b, rhs)
  ))
  ) input

and make_node ((b, rhs), lhs) = AST.Binop (b, lhs, rhs)

and prec_parse level input = (
  atom >>= (fn lhs =>
  many0 (prec_helper level) >>= (fn brhsl =>
  result (foldl make_node lhs brhsl)
  ))
  ) input

...
```

The LHS is parsed via the ```atom``` parser in the main ```prec_parse``` function. 
The while loop is converted to the helper parser ```prec_helper```
The condition is handled by the ```binop``` combinator, which takes the current level and returns a parser that matches on an operator (using the ```sigil``` parser, which parses sequences of special characters) if it has precedence >= current level. 
```binop``` is sequenced with the ```prec_parse``` parser, which corresponds to the recursive call in the imperative version. 
```many0``` returns at least zero repetitions of prec_helper into a list of results. 
```many0``` will thus parse as many operator-expression pairs as possible. If there were none, then the result is just the original atom parsed, because the behavior of ```foldl``` on an empty list is to return the initial item. 
Reassigning the LHS variable is taken care of by using ```foldl``` to properly nest the parsed operators and expressions into a binary operation AST node. 

The language has no top level yet (which is needed for declaring new infix operators) so I hardcoded basic arithmetic into the operator table. 
I won't be touching those aspects until later though as the top level needs support from the backend, which only supports the expressions. 

