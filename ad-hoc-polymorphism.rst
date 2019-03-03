##############################
Principled ad-hoc polymorphism
##############################

Part -1 - Before we start...
============================

When I say "Haskell", I mean GHC Haskell, not Haskell98 or Haskell2010 or
something else. I will also pretend that the Incoherent pragma and
IncoherentInstances extension do not exist for the sake of this post as they
are used very rarely.

A substantial portion of this post is based off *Modular implicits* [#f7]_,
which discusses various tradeoffs in the context of OCaml.

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
2. All type-related information that is accessed within a function body (either
   at compile time or at run time) is reflected in its type signature.

   In other words, principled ad-hoc polymorphism is similar to bounded
   parametric polymorphism. [#f1]_

Part 1 - Coherence
==================

There are several ways to go about adding some support for ad-hoc polymorphism
to that shiny type checker you're working on. A natural question to ask is:

  What are the possible options in the design space that have already been
  explored, and what are the ones being considered? What are the tradeoffs
  involved with each approach?

Before I can try to answer that question, we need to go over a key definition
[#f2]_ -

Coherence
    A type system is coherent if every different valid typing derivation
    for a program leads to a resulting program that has the same dynamic
    semantics. [#f3]_  [#f6]_

In looser wording, if there are multiple ways to assign types throughout the
program, all the ways lead to a program with the same runtime behaviour.
Whether you take the road less traveled by or not, it does not matter.

Notice that we just defined coherence *without* involving any kind of type
classes or traits or modules.

Hopefully, you will agree that coherence is a nice property to have; it
would be odd, if not downright nasty UX, to have the program run differently
based on what the type checker's mood is.

Part 2 - Canonicity
===================

(Note: The definitions I'm using in this section are not standard. If you know
of some standard definitions for the terms, e.g. from some textbook, please
submit a pull request!)

In Part 0, I said that style of ad-hoc polymorphism we are interested in here
is like bounded parametric polymorphism. Usually we'd like to be able to name the
bounds corresponding to something sensible. Consider the following pseudocode [#f4]_::

    bound Display (T : Type) {
        val display : T -> String
    }
    instance Display String {
        let display s = s
    }
    instance Display (List T) where (T : Type, Display T) {
        let display v = List.sepBy "," (List.map display v)
    }

Instances correspond to bound-type-constraint triples. In the first instance,
there is no constraint, whereas in the second instance, there is a constraint
that the type variable (a.k.a. generic parameter) ``T`` in the type
``List T`` also have an instance for Display.

Say you're working on an application and want to display some error message.
You might want the display the message differently to users, compared to the
display in the test suite or in logs. Are you allowed to have multiple instances
for ``Display MyErrorMessage``? This brings us to canonicity -

Canonicity
    At most 1 instance is compiled for a bound-type pair.

When canonicity is enforced, there are two kinds of instances that are of
particular interest:

Orphan instances
    Instances are usually placed with either the definition of the corresponding
    bound or the corresponding type; instances present in other modules
    (shorthand for "aggregations for definitions" here) are called
    *orphan instances*.

Overlapping instances
    If there are instances for which a substitution of type variables can lead
    to ambiguity on which instance can be picked, they are overlapping instances.

Different languages make different choices here.

* Haskell allows for orphan instances as well as overlapping instances.
  Note: Orphan instances are usually frowned upon and writing one gives
  you a warning with ``-Wall``.
* Purescript `forbids orphan instances
  <https://github.com/purescript/documentation/blob/master/language/Differences-from-Haskell.md#orphan-instances>`_.
  It uses `instance chains <https://github.com/purescript/purescript/pull/2929>`_
  for type class programming instead of overlapping instances.
* Rust does not allow orphan instances except in edge cases (see [#f5]_).
  Rust currently doesn't allow for overlapping instances, however there is a
  planned feature "impl specialization" [#f10]_ (available on rust 1.34.0-nightly)
  which allows specialized instances to override more general ones.

Part 3 - Problems with demanding canonicity
===========================================

Duplicate functions with implicit and explicit arguments
--------------------------------------------------------

In Haskell, Purescript and Rust, it is not uncommon for the standard library to
have duplicate versions of functions - one which implicitly takes arguments via
instances and another "_by"/"By" version which explicitly takes a
higher-order function instead. For example, Haskell's ``base`` library has::

    sort :: Ord a => List a -> List a
    sortBy :: (a -> a -> Ordering) -> List a -> List a

If you are a library author, offering just one kind of interface means that
library users need to potentially go through more trouble - if you offer the
implicit argument version, then the user might have to use newtypes to use
the API, and if you only offer an explicit argument version, then the caller
has to use the relevant instance functions themselves.

This also doesn't scale once the number of type variables or constraints
increases.

Workaround(s): None.

Non-modular development (orphans or overlap disallowed)
-------------------------------------------------------

If orphan instances are disallowed, and if package PB defines a bound B,
and another package PT defines a type T, then one must know of the other,
as a third party cannot write their own instance for the B-T pair.
This problem commonly crops up when (de)serialization and random generation
of values are done using bounds+instances.

Similarly, if overlapping instances are disallowed, then all the logic for
instance selection (via instance chains or otherwise) must be concentrated
in one place - and once that is handled, there is no wiggle room for
adding different logic in an external location (due to canonicity).

Workaround(s):

1. In the case where orphan instances are disallowed, one can either
create newtypes and write instances for them, or one can use feature flags (in
either PB or PT) to add instances. Neither of these are without their own
problems.

2. In cases where overlapping instances are disallowed, adding newtypes may
or may not be sufficient for writing new instances.

Troublesome Orphans (orphans allowed)
-------------------------------------

If orphan instances are permitted, then ensuring canonicity is more expensive,
as you need a global check to do so. This corresponds to a
long open GHC ticket `<#2356 https://ghc.haskell.org/trac/ghc/ticket/2356>`_.

If the corresponding checks are skipped (for performance reasons or otherwise),
one needs to be particularly careful around using orphan instances as the
compiler doesn't have your back.

Workaround(s): Be particularly careful around orphan instances.

Mental complexity with selection rules (overlap allowed)
--------------------------------------------------------

1. Understanding the rules as an end-user - In Haskell, overlapping instances
   do not work for matching instance heads. This is sometimes a source of
   confusion for someone trying out overlapping instances
   (`<SO 1 https://stackoverflow.com/a/16776676/2682729>`_,
   `<SO 2 https://stackoverflow.com/a/30707290/2682729>`_).

2. Possibly unexpected "time travel" - code written in a compilation unit can
   change the semantics of code written its *dependencies*::

     module M1 {
         bound Display (T : Type) {
             val display : T -> String
         }
         instance Display (List T) where (T : Type, Display T) {
             let display v = sepBy "," (map display v)
         }
         val display_m1 : Display T => List T -> String
         let display_m1 v = display v
     }

     module M2 {
         import M1 (..)

         type X = MkX
         instance Display X        { let display _ = "Generic." }
         instance Display (List X) { let display _ = "Special." }

         let main () =
           print (display [MkX]);
           print (display_m1 MkX])
     }

     M2.main ()

   The equivalent Haskell program prints "Special.Generic." with
   ghc 8.6.1 (`manual <https://downloads.haskell.org/~ghc/8.6.1/docs/html/users_guide/glasgow_exts.html#overlapping-instances>`_)
   (giving up coherence), whereas the equivalent Rust program [#f11]_
   prints "Special.Special." with rustc 1.34.0-nightly
   (giving up separate compilation).

3. Working with weaker assumptions in the type-checker - One needs to consider
   all the possible cases where overlap might affect typing judgements and be
   careful about edge cases. This becomes more complicated when more features
   like associated types/associated type families are thrown into the mix.

Workaround(s): For the "time travel" problem, one could use dynamic dispatch
(like virtual functions in OO systems) to have both coherence and separate
compilation at the cost of reduced efficiency.

Zero cost newtypes are tricky
-----------------------------

Newtypes are not always zero cost. If you have a vector of newtypes, then
you can't convert it to a vector of the underlying type with no allocations
in a type-safe manner unless you explicitly add in type-safe coercions.

Haskell adds support for type-safe coercions via the ``Coercible`` type class.
However, this adds a non-trivial amount of complexity to the compiler
implementation (look up GHC's role system if you're interested in the details),
and creates more room for bugs.

Workaround(s): Use unsafe coercions in a controlled manner.

Part 4 - Non-canonicity
=======================

As you might already know, not all languages enforce canonicity. Scala's
implicit objects and OCaml's modular implicits are two examples
where coherence is maintained but canonicity is not.

We can still talk about orphans and overlap here -

* Scala allows for orphan instances as well as overlapping instances. In case
  of overlap, there are some precedence rules that determine which instance is
  picked.
* OCaml allows for orphan instances but not overlapping instances.

Part 5 - Problems with eschewing canonicity
===========================================

Types need to keep track of corresponding instances
---------------------------------------------------

Note: This is sometimes called the "Set Ordering problem" or the "Hash table
problem".

If canonicity is guaranteed, the signature of a set union function will look
like::

    val union : forall a. Ord a => Set a -> Set a -> Set a

However, in the absence of canonicity, this function cannot be implemented as
the two sets might have been created with different ``Ord`` instances.

OCaml solves this problem using parameterized modules (in the absence of
implicits). ``Set`` acts as a parameterized module that takes in an ``Ord``
module (say ``IntAscendingOrd``) and creates another module
(say ``IntAscendingSet``) with a set type and functions (such as ``union``)
which operate on that type. If you apply ``Set`` to another ``Ord`` module
(say ``IntDescendingOrd``), that creates a distinct module (say ``IntDescendingSet``).
``IntAscendingSet.union`` cannot operate on values of type ``IntDescendingSet.t``,
so crisis is averted. This strategy carries over to implicits in a
straight-forward manner. Hence, the signature of ``union`` becomes::

    val union : {O : Ord} -> Set(O).t -> Set(O).t -> Set(O).t

Workaround(s): With a little bit of practice (you must educate your users!),
you can get the hang of writing APIs without such soundness issues.

Less well-studied in mainstream languages compared to type classes
------------------------------------------------------------------

If you want to implement type classes, you can read the various papers related
to Haskell's type classes, as well as play around with the implementations in
Haskell, Purescript, Rust (traits), Swift (protocols) and more.

On the other hand, if you want to forego canonicity - the two mainstream
languages that you can study are Scala and OCaml. [#f16]_ Moreover, the
OCaml implementation is very much work-in-progress (there is a prototype
though); there isn't a paper with the technical details of type checking for it
yet.

Workaround(s): None.

Path-independence proofs (alt. diamonds are a programmer's worst enemy)
-----------------------------------------------------------------------

1. Picking instances through different paths - This is very similar to the
   problem that arises in the presence of multiple inheritance. For example,
   say Traversable and Monad are two bounds that depend on Functor (either
   via composition or via inclusion/inheritance, it doesn't matter)::

     -- Arguments that come before '=>' are implicit
     val map : Functor f => (a -> b) -> f a -> f b
     let map Ff f x = Ff.map f x

     val contrived_map : Traversable f => Monad f => (a -> b) -> f a -> f b
     let contrived_map Tf Mf f x = map     g x
                                     --^^^--
                Should we get Functor from Traversable or Monad?
                How do we assert that both of them should be the same
                in the absence of canonicity?

   While the above example is contrived (for brevity), such a situation
   comes up very frequently in Haskell code, where ``Monad`` is a common
   subclass across several different constraints. [#f14]_

2. Deriving instances through multiple paths - If bounds/instances
   utilize/permit subtyping, then derived instances can lead to ambiguities::

     instance Eq (List T) where (T : Type, Eq T) { .. }
     instance Ord (List T) where (T : Type, Ord T) { .. }
     -- instance Eq Int is unnecessary as we can up-cast the Ord instance
     instance Ord Int { .. }

     let _ = equal     [1, 2] [2, 3]
                 --^^^--
         Should we create Ord (List Int) and then up-cast to Eq (List Int)
         or up cast (Ord Int) to Eq Int and then create Eq (List Int).

   This problem can be avoided in multiple ways:

   a. Using composition so up-casting doesn't work. So you'd need to define
      ``Eq Int`` separately and only that gets used to create
      ``Eq (List Int)``.

   b. Getting rid of implicit up-casting for implicit arguments. Again, you
      can manually write ``Eq Int``, which gets used to create
      ``Eq (List Int)``.

Workaround(s): To solve problem 1, in Scala, one can use composition + implicit
conversions. This is called the Scato encoding [#f13]_. In my opinion,
this approach cannot work for large hierarchies as the boilerplate required
scales super-linearly. On a more fundamental level, what we want to be able to
say is "prove that these N derivations lead to the same outcome and give me a
type error if that isn't the case", whereas the Scato encoding says "here's
the most privileged derivation, please prefer it over other derivations".

Speculation: It might be possible to have a module system with a merge operation
where the merge succeeds only if the common exports (types and terms) are
provably equal [#f12]_. Equations are propagated through applicative functors,
transparent ascription, manually written equalities and common inclusions.

No inference for implicit parameters
------------------------------------

Haskell and Purescript support inference of constraints which can be helpful
if you're not exactly sure on what's needed - you can ask the compiler to fill
in the details for you. Implicit parameters cannot be inferred in Scala or
OCaml, which can be disappointing.

Workaround(s): None.

Part 5 - Problems independent of canonicity
===========================================

Nominal vs structural bounds
----------------------------

Bounds usually have a "subclassing" mechanism associated with them. If bounds
are nominal, this means the hierarchy cannot be modified without modifying all
the code dependent on it. For example, Haskell originally didn't have Applicative
as a superclass of Monad (and Semigroup as a superclass of Monoid), so adding the
superclasses were breaking changes. Historical artifacts of this can be
seen today in the form of redundant functions (``return`` and ``pure``,
``*>`` and ``>>``). The ``semigroupoids`` package contains additional classes
Apply and Bind, but these cannot be treated as superclasses of Applicative
and Monad respectively.

Structural bounds don't suffer from these issues - users are free to "project
out" subsets without worry, if one uses subtyping via inheritance/inclusion.
However, subtyping comes with its own set of problems as discussed earlier.

Part 6 - Closing thoughts
=========================

Debates and discussions on type classes vs modules (or other things) often
end up re-iterating the same points in several distinct ways. Newer systems with
traits/protocols etc. complicate the design space further (at least,
superficially). Formulating the question of "What system do I use?" in
terms of key desirable properties and resulting incompatibilities is a useful
exercise that highlights the commonalities and distinctions between different
systems.

No points for guessing that there isn't a clear "right answer" here.
Middle-of-the-road solutions such as in
`Dotty #2047 <https://github.com/lampepfl/dotty/issues/2047>`_
might be worth exploring in other contexts too.

Lastly, there are a couple of references that I intended to talk about but
didn't. You might still want to look at those. [#f8]_  [#f9]_

.. [#f1] You might ask: why don't I just use the term "bounded
         parametric polymorphism" instead of "principled ad-hoc polymorphism"
         then? The answer is that we will usually be interested in having some
         fine-grained way of controlling instances, so parametricity won't be quite
         applicable once we do that.

.. [#f2] `Type classes: confluence, coherence and global uniqueness
         <http://blog.ezyang.com/2014/07/type-classes-confluence-coherence-global-uniqueness/>`_ -
         Edward Z. Yang

.. [#f3] `Type classes: an exploration of the design space
         <https://www.microsoft.com/en-us/research/wp-content/uploads/1997/01/multi.pdf>`_ -
         Simon Peyton Jones, Mark Jones and Eric Meijer, 1997.

.. [#f4] The terminology varies from language to language :(. Haskell and
         Purescript use "type class" and "instance", Rust uses "trait" and "impl",
         Swift uses "protocol" and "extension". I've chosen "bound" as there is
         a direct connection with bounded quantification.

.. [#f5] `Rust RFC 1023
   <https://github.com/rust-lang/rfcs/blob/master/text/1023-rebalancing-coherence.md>`_ -
         Niko Matsakis, Aaron Turon and Alex Crichton.

.. [#f6] It *seems* the Rust community uses the same terminology to mean
         something different here (although as Edward Yang's blog post shows,
         there was enough confusion about it in the Haskell
         community at some point to warrant a blog post). If you look at the
         explanation for `Orphan Rules <https://youtu.be/AI7SLCubTnk?t=3027>`_
         or Rust RFC 1023 [#f5]_ or the
         `Little Orphan Impls <http://smallcultfollowing.com/babysteps/blog/2015/01/14/little-orphan-impls/>`_ blog post,
         the Rust community uses the term "coherence" akin to Yang's
         "global uniqueness of instances" or more succinctly
         "canonicity" (this post and [#f7]_).

         Similarly, Martin Odersky's comments in the context of Scala
         (`Dotty #4234 <https://github.com/lampepfl/dotty/issues/4234#issue-310774757>`_,
         `Dotty #2047 <https://github.com/lampepfl/dotty/issues/2047#issue-211652526>`_)
         seem to use the term "coherence" as shorthand for "uniqueness of
         instances".

.. [#f7] `Modular implicits <https://arxiv.org/pdf/1512.01895.pdf>`_ -
         Leo White, Frédéric Bour and Jeremy Yallop.

.. [#f8] `Type classes vs. the World [video]
         <https://www.youtube.com/watch?v=hIZxTQP1ifo>`_ -
         Edward Kmett.

.. [#f9] `scala/dotty PR#5458 - An Alternative to Implicits
         <https://github.com/lampepfl/dotty/pull/5458#issue-231614409>`_ -
         Martin Odersky.

.. [#f10] Impl specialization:
          `PR <https://github.com/rust-lang/rust/issues/31844>`_,
          `RFC 1210 <https://github.com/rust-lang/rfcs/blob/master/text/1210-impl-specialization.md>`_.

.. [#f11] Writing an equivalent Rust version is a bit tricky as Rust doesn't
          allow orphan instances (for writing the impl for ``Vec<T>``). So the
          trait needs to take an extra parameter to get the same effect.

.. [#f12] This would be similar to MixML's [#f15]_ ``with`` operation
          except that MixML disallows common exported terms.

.. [#f13] `The Limitations of Type Classes as Subtyped Implicits(Short Paper)
          <https://adelbertc.github.io/publications/typeclasses-scala17.pdf>`_ -
          Adelbert Chang

.. [#f14] I haven't written enough Purescript or Rust code to say the same there.

.. [#f15] `Mixin' up the ML module system
          <https://storage.googleapis.com/pub-tools-public-publication-data/pdf/43982.pdf>`_ -
          Andreas Rossberg and Derek Dreyer.

.. [#f16] Brendan Zabarauskas `points out
          <https://www.reddit.com/r/ProgrammingLanguages/comments/awoi1d/principled_adhoc_polymorphism/eho42r0>`_
          that Idris interfaces, Agda's instance arguments, Coq type classes
          and Lean type classes all do not force canonicity.
