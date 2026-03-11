# Lab 02: Lexer

This assignment is the first stage of the Amy compiler.

## Code Scaffold

In this lab you will start your own compiler from scratch, meaning that you will no longer rely on the compiler frontend which was previously provided to you as a jar file. In this lab you will build the lexical analysis phase (`lexer`).

As we are now starting to work on your own compiler, we will start with a fresh scaffold. We suggest to keep the interpreter alongside, and we will tell you at what point you can add back your interpreter in your project.

For now, you should work in this new project that will become your full compiler. Following labs will be delivered as files to add to this project.

The structure of your project `src` directory should be as follows:

```text
lib 
 ├── scallion-assembly-0.6.1.jar
 ├── stainless-library_3-0.9.9.2-6-g78f42ac-SNAPSHOT-sources.jar
 └── ziplex_3-0.1.0-SNAPSHOT.jar
 

library
 ├── ...
 └── ...

examples
 ├── ...
 └── ...

amyc
 ├── Main.scala                         
 │
 ├── parsing                             
 │    ├── Lexer.scala
 │    └── Tokens.scala
 │
 └── utils                               
      ├── AmycFatalError.scala
      ├── Context.scala
      ├── Document.scala
      ├── Env.scala
      ├── Pipeline.scala
      ├── Position.scala
      ├── Reporter.scala
      ├── UniqueCounter.scala
      └── ZiplexUtils.scala

test
├── scala
│    └── amyc
│         └── test
│               ├── CompilerTest.scala
│               ├── LexerTests.scala
│               ├── TestSuite.scala
│               └── TestUtils.scala
└── resources
      └── lexer
           └── ...

```

This lab will focus on the following two files:

* `src/amyc/parsing/Tokens.scala`: list of *Amy* tokens and token kinds.
* `src/amyc/parsing/Lexer.scala`: skeleton for the `Lexer` phase.

Below you will find the instructions for the second lab assignment in which you will get to know and implement an lexer for the Amy language.

## A Lexer for Amy

The role of a lexer is to read the input text as a string and convert it to a list of tokens. Tokens are the smallest useful units in a source file: a name referring to a variable, a bracket, a keyword etc. The role of the lexer is to group together those useful units (e.g. return the keyword else as a unit, as opposed to individual characters e, l, s, e) and to abstract away all useless information (i.e. whitespace, comments).

## Code structure

You can find the `lexer` in the `Lexer.scala` file. It is based on ZipLex, a formally verified lexer Scala library. Ziplex allows you to transform an input character stream (such as the contents of an Amy source file) into a sequence of Tokens. We include a short reference of Ziplex's API in `Lexer.scala`.

To build a lexer using Ziplex, you first define *rules* based on regular expressions. Each rule is associated with a transformation function that converts the matched input characters into a `TokenValue`. Ziplex then offers a `lex` function that takes as input the list of your rules along with the input string and produces a sequence of tokens.

`Ziplex` follows what is known as the **longest match** semantics (or **maximum munch**): at each step, it finds the rule that matches the longest prefix of the remaining input to create a token. If multiple rules match the same longest prefix, the rule that appears first in the list of rules is chosen.

The `TokenValue` classes and the transformation functions from matched strings to `TokenValue`s are defined in `ZiplexTokens` in `Lexer.scala`. These are provided but you might want to see how they work.

Ziplex's `lex` function returns a sequence of `ziplex.Token`s, whose class definition is the following:

```scala
case class Token[C](value: TokenValue, rule: Rule[C], size: BigInt)
```

Here, `value` is the `TokenValue` produced by the transformation function of the matched rule, `rule` is the rule that matched the input, and `size` is the length of the matched input.

The rest of Amy's pipeline expects tokens of type `amyc.parsing.Tokens.Token`. Therefore, the final step of the `Lexer` is to convert the `ziplex.Token`s into `amyc.parsing.Tokens.Token`s. This is done in the `toAmyToken` function in `Lexer.scala`. Some cases are already implemented for you, but you will need to complete it.

Positions are handled for you by the `run` method of the `Lexer`. The `ziplex.Token`s returned by `Ziplex` do not contain position information, but since they contain the size of the matched input, it is possible to reconstruct the position of each token in the input string. The `run` method does this and sets the position of each token before returning it. You can have a look at `ZipLexUtils.nextPosition` and `ZipLexUtils.addPositions` in `ZiplexUtils.scala` to see how this is done.

The `Lexer` has the following components:

* The public method is `run`. It calls `ZipLexUtils.lex(rules, input)` for every input file, add positions of tokens, transforms them to Amy tokens, and concatenates the results. For each file, the `run` method inserts an `EOFToken` at the end of the token stream.
* All the rules should go in the list `rules`. You will need to complete this list by adding all your rules in the right order of priority.
* Each rule is defined as a `val` in `AmyLexer`. You will need to complete the definitions of these rules.
* Whenever a rule is found to match a (maximal) prefix of the remaining input, Ziplex produces a token, then continues on lexing the rest of the input.
For more details on how to write new rules, read the short introduction to Ziplex's API at the top of `Lexer.scala`.

Your task is to complete the rules in `Lexer.scala` and implement the filtering of irrelevant tokens.

## Notes

Here are some details you should pay attention to:

* Make sure you recognize keywords as their own token kind. `if`, for instance, should be lexed by the keyword rule, not as an identifier with the content `if`.
* In general, it is good to output as many errors as possible (this will be helpful to whomever uses your compiler, including yourself). There are certain inputs that you might explicitly want to map to `ErrorToken`, such as unclosed multi-line comments, or out of bound integer literals. You can for example do it in the `toAmyToken` function. The `run` method will call `ctx.reporter.fatal` on any `ErrorToken` it encounters.
* The Lexer returns an `Iterator`[`Token`] that will be used by future phases to read tokens on demand.
* Comments and whitespace should not produce tokens. The most convenient way is to let Ziplex produce them, and filter them out in `toAmyToken`. See the related `TODO` in `Lexer.scala`.
* Returned Amy tokens should be fresh instances of the the appropriate Token subclass. Value tokens (tokens that carry a value, such as identifiers), need to be constructed with the appropriate value.
* Make sure to correctly implement the Amy lexing rules for literals and identifiers.

## Example Output

For reference, you can look at resources in the test folder to see example outputs.

## Deliverables

Deadline: **Deadline: **12.03.2026 23:59:59****.

As for the previous lab, you should submit your work on the corresponding Moodle assignment. You should submit the following file:

* `Lexer.scala`: your implementation of the lexer.
