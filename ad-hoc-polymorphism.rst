##############################
Principled ad-hoc polymorphism
##############################

Part 0 - What the heck does the title mean?
===========================================

Quoting the Haskell wiki:

  Ad-hoc polymorphism refers to when a value is able to adopt any one of several
  types because it, or a value it uses, has been given a separate definition for
  each of those types. For example, the + operator essentially does something
  entirely different when applied to floating-point values as compared to when
  applied to integers – in Python it can even be applied to strings as well.

Okay, that doesn't sound very "principled". So what is up with "principled"
in the title? Well, "principled" for the sake of this article means 2 things -

1. Sub-expressions can be assigned types. As a counter-example, in C++, if you
   had a function ``plus`` with the overloads as for Python, it is not
   meaningful to speak about the type of ``plus``; you can only speak about
   the types of the individual overloads themselves.
2. Type signatures are fully honest. As a counter-example, if the language
   allows code inside a function to explicitly access type information
   (either at compile time or run time) without reflecting it in the types
   signature of the function itself, then it is not honest.

   In other words, ad-hoc polymorphism should act like bounded parametric
   polymorphism. [#f1]_

Part 1 - Ground rules
=====================

There are several ways to go about adding some support for ad-hoc polymorphism
to that shiny type checker you're working on. A natural question to ask is:

  What are the possible options in the design space that have already been
  explored, and what are the ones being considered? What are the tradeoffs
  involved with each approach?

Before I can try to answer that question (the "try to" is necessary, as I'm
hardly familiar with a sliver of the design space myself), we need to go over
a key definition [#f2]_ -

Coherence
    A type system is coherent if every different valid typing derivation
    for a program leads to a resulting program that has the same dynamic
    semantics. [#f3]_

In looser wording, if there are multiple ways to assign types throughout the
program, all the ways lead to a program with the same runtime behaviour.
Whether you take the road less traveled by or not, it does not matter.

Notice that we just defined coherence *without* involving any kind of type
classes or traits or modules.

   **Exercise:** Consider a very simple typed language with integer literals,
   integer comparison and boolean literals that works as you might expect. Say
   we add an ``if`` construct (which evaluates in the usual manner) with the
   following typing rules::

     If x : X then (if True x y) : X
     If y : Y then (if False x y) : Y
     If x : X and y : X and b : Bool then (if b x y) : X

   Is the type system coherent?

Hopefully, you will agree that coherence is a nice property to have; it
would be odd, if not downright nasty UX, to have the program run differently
based on what the type checker's mood is. [#f4]_

Part Last - References + suggested reading
==========================================

If you'd like to learn more about type systems formally, I recommend starting
with `Types and programming languages <http://www.cis.upenn.edu/~bcpierce/tapl/>`_
(a.k.a. TAPL).

.. [#f1] You might ask: why don't I just use the term "bounded
         parametric polymorphism" instead of ad-hoc polymorphism then? Well, the
         simple answer is that it is more verbose.

.. [#f2] Thanks to Edward Z. Yang for his blog post:
         `Type classes: confluence, coherence and global uniqueness
         <http://blog.ezyang.com/2014/07/type-classes-confluence-coherence-global-uniqueness/>`_.

.. [#f3] `Type classes: an exploration of the design space
         <https://www.microsoft.com/en-us/research/wp-content/uploads/1997/01/multi.pdf>`_
         - Simon Peyton Jones, Mark Jones and Eric Meijer, 1997.

.. [#f4] It seems the Rust community uses the same terminology to mean
         something different here (although as Edward Yang's blog post shows,
         there was (still is?) enough confusion about it in the Haskell
         community at some point to warrant a blog post). If you look at the
         explanation for `Orphan Rules <https://youtu.be/AI7SLCubTnk?t=3027>`_
         or Rust RFC 1023 [#f5]_, the Rust community uses the term "coherence"
         akin to Yang's "global uniqueness of instances" or more succinctly
         canonicity (this note).

.. [#f5] `Rust RFC 1023
         <https://github.com/rust-lang/rfcs/blob/master/text/1023-rebalancing-coherence.md>`_ -
         Niko Matsakis, Aaron Turon and Alex Crichton.

.. [#f6] `Modular implicits <https://arxiv.org/pdf/1512.01895.pdf>`_ -
         Leo White, Frédéric Bour and Jeremy Yallop.

.. [#f7] `Type classes vs. the World [video]
         <https://www.youtube.com/watch?v=hIZxTQP1ifo>`_ -
         Edward Kmett.

.. [#f8] `scala/dotty PR#5458 - An Alternative to Implicits
         <https://github.com/lampepfl/dotty/pull/5458#issue-231614409>`_ -
         Martin Odersky.
