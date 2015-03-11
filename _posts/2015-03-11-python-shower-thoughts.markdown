---
layout: main
title: Python Shower Thoughts
---

# Python Shower Thoughts

#### I use unit tests as a compiler

Whenever I change my Python code, even really slightly (like modifying a string literal or removing an unused import), I re-run the test suite before committing.  Isn't that sort of weird?

I've written (personal) projects sans unit tests before, when I didn't know they existed - my third- and fourth-year university projects were both done with very few unit tests. By and large, this was mostly OK. I tested them manually a lot, separated out my concerns as much as possible in typical OOP fashion, and used logging and Visual C++'s step-through debugger when the code exploded or displayed aberrant behaviour.

In retrospect, I think this only worked at all because my IDE and the C++ compiler were two gigantic safety nets.  The compiler caught straightforward syntax errors (which Visual C++ can helpfully flag as I type, so I didn't even have to run the compiler to check my code), and the combination of the two gave me good error output when runtime errors occurred.  The bugs I spent the longest on were ones unit tests probably wouldn't have helped solve, such as weird rendering errors brought forth by slightly incorrect logic.  I don't think I could do the same thing in Python.

SublimeText plugins and the various available linters can only go so far in catching typos, incorrectly named variables, type errors, and so on.  Python's incredibly permissive data storage systems make this more of an issue, because type errors often go unnoticed and blow up something else further down the line (or worse, just give you weird output), and there's no good way for a linter to tell if and when your `User` class actually contains a field called `email`.  And so we need unit tests - an extensive suite of basic sanity checks that catch typos as much as they do logic errors.  Something that will kindly blow up if you make an `ImportError` without you having to commit or manually click through the locally-running website, saving you shame and faffing respectively.

In C++ land, and everywhere else, that's called a compiler.


#### Are pointers genuinely that scary?

"Everything in Python is a dictionary."

"Well, no - everything in Python is an object."

Well, actually, everything in Python is a pointer, but given how much people seem to avoid writing about that, it might as well be called The Type Which Must Not Be Named.

I remember reading an article that explained in-depth how Python's names and objects work.  (Sadly, the link has been lost to time.)  It was a pretty good writeup, but I found myself sitting there asking 'why?' at every bizarre-looking design decision.  Why on earth does `a = 1; b = a; b = 2` set a to 2?  That just seems like an unnecessary pitfall that you'll almost never use as a feature.  Why would you make strings always immutable?

About three-quarters of the way through the article a big lightbulb went on my head and I said, out loud if I recall, "Oh! They're pointers!".  Because Python is built on top of C, it makes sense that it wouldn't have primitives - each data item in Python, be it a simple number literal or a huge data structure, is going to be represented by a C struct, probably a large and complex one at that.  That struct will have memory allocated to it and be referred to by a pointer.  It's much more efficient to just do `a = b` in the C engine when the Python programmer writes the same thing, especially if half your objects are immutable anyway and will be copied if the programmer wants to operate on them.

I asked a coworker and friend (who's written Python almost exclusively, and knows it in substantial depth) about this, and his reply was along the lines of "the term 'pointer' has baggage".  The idea, at least as I understood it, is that pointers are an element of a different language that are also widely considered to be complicated or evil, so Python aims to abstract away from them.  This makes sense on the surface, but Python exposes more than enough pointer-y behaviour for it to look utterly bizarre and convoluted if you don't understand _why_ names work the way they do.

Realising that Python represents absolutely everything via a pointer explains a lot of its stranger "design decisions".  Meta-programming, reassigining `__str__` and other double-underscore methods, and similar trickery seems weird and mostly unnecessary, but it was probably easier for Python to allow its users to take advantage of its internal flexibility if they needed it, given all of those names are presumably backed by pointers that can be straightforwardly changed.  Less work for more power?  Wouldn't you?

I never found pointers hard, myself.  They're certainly less complicated when you're using them explicitly than Python's metaprogramming systems are, and knowing about them and how they link into Python has considerably boosted my understanding of Python.  The tutorial that explained everything to me was [this one, from the excellent cplusplus.com](http://www.cplusplus.com/doc/tutorial/pointers/), and I hope it helps somebody else in the same way!  It's obviously written using C++ syntax, but I don't think there's anything grossly complicated in there.  (The first few sections are the most relevant ones, and the later bits are more in-depth and C++-specific.)


#### A test isn't a real spec

Test-driven development, or TDD, is fairly popular.  The idea, for those unfamiliar, is that you write the unit tests for a particular piece of code _before_ the code itself, and write your code to make the tests pass.  You can break it down into smaller steps by writing a test that describes only a little piece of the functionality you want, making that pass, extending it, making that pass, and so on.  It works well because it forces you to explicitly specify to yourself what you're doing, similarly to writing a bunch of comments before you start programming.  I don't use it much myself because my brain prefers to try stuff and see where it goes rather than make all of the decisions up-front, but I know people who really like it and I've used it enough to assert its effectiveness.  So far so good.

However, I've taken over a handful of unfinished [pull requests](https://help.github.com/articles/using-pull-requests/) over time from coworkers who've been moved off a project before they could get properly started, leaving only the beginnings of a task.  Sometimes, those beginnings are just a test or three, because they've been writing in the TDD style.  And I find those pull requests surprisingly difficult to pick up.

The main reason behind this is that you can't actually write a TDD test without some idea of what the implementation will look like.  For trivial tasks, that's OK since someone else will have the same idea, but your test is still written describing the implementation as much as the test.  Only tests on the level of [user stories](http://www.mountaingoatsoftware.com/agile/user-stories) - like "this test will pass if the user can save their work" - can avoid this, and if your unit tests look like user stories you have another problem.  Unit tests are meant to be small and granular to facilitate easier debugging.  On top of that, unless your TDD tests happen to be a complete description of the spec and implementation (which they will be if you've finished the pull request, but they probably aren't in this instance), the person picking up your work is going to have to look up the spec anyway, to see what they're supposed to be doing.

Then, once I've looked up the spec for a problem, I might as well solve it in my own way.  If you and I have the same thought process, that's not an issue - your test will mostly work, and will save me time writing it myself.  If we take different approaches, I might have to scrap or heavily modify what you've written.  If I try and reconstruct what you were doing, I have to try to read between the lines of your test to determine what a piece of your intended implementation looks like, and then extrapolate from there.

The other issue is that tests are, well, code, and if you can't run them (or are expecting them to fail) then you have no way of knowing that they're correct.  Running unit tests checks your tests against your code just as much as it checks your code against your test, which is something I think people forget.  I've spent a comparable amount of time debugging my tests and debugging the code they're meant to check, which isn't massively reassuring.  This means that when you look at a test, you have no real way of knowing how closely you can trust it as a specification - it might be wrong, it might be broken, it might not have passed code review (when you're picking up an unfinished pull request), it might not actually match the real spec.  Obviously, my coworkers are supremely competent people (and more experienced Python developers than I am), but I have a habit of not trusting tests any more than I would the code they're checking, and we all know that even the best programmers produce bugs.  Logically, they produce buggy tests, too.

This is a really minor problem, of course - 99.5% of TDD, at least in my office, is immediately followed by a nicely-written solution.  It's extremely illuminating, though.  I've heard the idea of "tests as specification" floating around the internet or programmer chat a few times, and it kind of unnerves me.  I get the idea, but I'd personally much rather work from a well-written English description of what I'm supposed to be doing, or review code based on how it solves a written problem description rather than try to puzzle out what it's doing by reading its tests.  (Yes, I'm one of those "comments are useful" types.)  For all the imprecision of natural language, it tends to contain far fewer bugs than code, even well-written code.  If nothing else, the machine interpreting your typos will raise an eyebrow and understand you anyway rather than explode.
