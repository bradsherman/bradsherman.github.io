---
layout: post
title: My Thoughts on Writing an Interpreter in Go
---

One of my favorite classes in college was Compilers. The class consisted
mainly of a semester long project that involved building a compiler (in
C) for a high level C-like language called C-Minor. Something about
knowing how the source code I type in is translated into something
machines can understand really intrigued me. However, as I'm sure many
other college students can relate, I did not have the time to give this
class as much attention as I would have liked given the rest of my
workload.

Fast forward to today, and I think it would be cool to write my own
interpreter again. In my
first post I mentioned how I enjoy doing projects to learn a language
after I have read through an introductory book. I decided I'd like to
write an interpreter and/or compiler in Rust but once I started thinking
about it none of the things I originally learned from my Compilers
class really stuck which is a bummer! Thankfully, right around this
time I stumbled upon [Writing an Interpreter in Go](https://interpreterbook.com/)
by Thorsten Ball. It was exactly what I was looking for! A practical
way to understand the fundamental concepts of lexing, parsing, and
evaluating without getting too bogged down in the theory that many
compilers textbooks come with. So, I decided to purchase Writing an
Interpreter in Go and get started!

In an attempt to avoid forgetting all the information (again), I decided
to write this blog post explaining the high level concepts of each
section. Teaching something is much more difficult than following along
in a book, so I am hopeful that this exercise will solidify the
concepts. It is important to note however, that I will sacrifice some
detail for the sake of brevity. My secondary goal is sparking your
interest enough that you'll grab the book for yourself!

## Lexing

In my opinion, the easiest part of this whole experience is writing the
lexer. A lexer basically is responsible for reading in the source code
 and spitting out tokens. Tokens are small units of data with a
name and a type that are categorized and fed to the parser, which I'll
touch more on later. For example, in a lot of languages the token ";"
signifies to the parser that it has reached the end of a statement. To
make this even more concrete I will give a small example. Consider the
following line of Rust code:

{% highlight rust %}
let x = 4 + 2 * 4;
{% endhighlight %}

The Rust lexer most likely reads this line of code into the following sequence of tokens:

{% highlight markdown %}
TOKEN_LET "let"
TOKEN_IDENT "x"
TOKEN_ASSIGN "="
TOKEN_INT "4"
TOKEN_PLUS "+"
TOKEN_INT "2"
TOKEN_ASTERISK "*"
TOKEN_INT "4"
TOKEN_SEMICOLON ";"
{% endhighlight %}

(or something similar, the exact names don't matter). Notice how the
spaces in between each character were ignored! This is how Rust works,
but in other languages like Python white space is more important and
will be read in as `TOKEN_SPACE` or `TOKEN_TAB`.
These tokens will then be used by the parser to determine what structure
is taking shape in the code. In this case, it's a variable assignment.
Like I said lexing is pretty simple, but the next
step is where things get a little more complicated.

## Parsing

The next step is to implement a parser. The parser is responsible for
consuming the sequence of tokens the lexer spits out and returning a
data structure (frequently referred to as an [Abstract Syntax Tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree)
or AST) that represents the original input. As Ball explains in the
book, parsing is one of the more well-understood problems in computer
science, and there are many different approaches to parsing. In fact,
there exists software capable of generating your parsers that implement
these approaches based on a set of rules defining your language (called
the "Grammar" of the language). Explaining the different types of
grammars and their corresponding parsing algorithms is outside the scope
of this post, and can easily be found in many of the canonical
textbooks on compilers so I won't spend time going over them.

In any case, in order to learn more about how parsing works, we hand
write our parser, specifically a "top down operator precedence" parser
(also called a "Pratt parser", named after Vaughan Pratt). This mostly applies to how expressions
are parsed, as that is the most complicated part of parsing. Ball does a
great job of distilling the main ideas behind the Pratt parsing
methodology, and there are other good references on the topic such as
[Pratt's orginal paper](https://tdop.github.io/) and [this](http://journal.stuffwithstuff.com/2011/03/19/pratt-parsers-expression-parsing-made-easy/)
post by Bob Nystrom. For those reasons, I won't get into the details of how
top down operator precedence works. Instead, I'll just walk through an
example of what the parser does using our original line from above:

{% highlight rust %}
let x = 4 + 2 * 4;
{% endhighlight %}

The parser will encounter this line represented as the sequence of tokens
above, starting with the `TOKEN_LET`. The point of the parser is to turn
those tokens into a data structure, the AST, that in this case might look
something like this:

{% highlight text %}
         +
        / \
       4   *
          / \
         2   4
{% endhighlight %}

However, this is not trivial! An alternative tree could be constructed from
our sample code that looks like this:

{% highlight text %}
         *
        / \
       +   4
      / \
     4   2
{% endhighlight %}

If we evaluate these trees from the bottom up (more on this below), we get 12
and 24, respectively. Obviously, one of these trees are wrong. The question is,
how can we write code that will always produce the correct tree?

That is the essence of parsing! Turning tokens into a data structure that is
easier to work with. This has been compared many times to building a sentence
out of words. Once we have the AST we can do all sorts of things such as
optimize away dead code, expand macro calls, and of course evaluate the code!

## Evaluating

flesh out

Now that we are able to tokenize the source code and product an AST, the
final step is evaluation. In its simplest form, evaluation simply means
walking the AST and doing what each node represents. This could be invoking
a function call with a list of parameters, or simply performing multiplication
and addition like our example above.

## What's next?

talk about the book a bit more

My original goal in reading this book was to reintroduce myself to the main
concepts involved in creating an interpreter or compiler. I've tried to keep this
post fairly high level, but hopefully it encourages you to try learning how to
write your own interpreter! Now that I've re-familiarized myself with the
fundamental concepts of interpreting, I plan on taking the next step and reading
[Writing a Compiler in Go](https://compilerbook.com/), also by Thorsten Ball!
