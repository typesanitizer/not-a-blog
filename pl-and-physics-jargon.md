# Programming language and physics jargon - a coincidence?

(I'm intentionally being loose with terminology here, don't be mad.)

When talking about variables or type variables (a.k.a. generic parameters) in a
programming language, here are some terms that are commonly used:

* binder - A syntactic construct that introduces a variable (into scope).

* bound variable - A variable that has an associated binder. If you've learned
  Racket or OCaml, you might've been confused by ["unbound identifier"](https://stackoverflow.com/q/32186599/2682729)
  or ["unbound module"](https://stackoverflow.com/q/13694140/2682729) errors at
  some point.

  I didn't really have a solid intuition for this, why is the variable "bound"
  -- is the binding "holding it" in some way? Doesn't seem to make much sense.

* free variable - A variable that is not bound.
  This makes sense to me.

* scope - There are at least two ways to think about this. One way is that a
  scope is a set of bindings. Another way to think about it as a region of
  source code that surrounded by boundaries on two sides (often literally).

  I think you replace the word "scope" with "boundary", the usage of the term
  "bound variable" makes a lot of sense.
  Unfortunately, the word "boundary" is already used loosely in several places,
  e.g. "API boundary", "module boundary", "function boundaries", so it is tricky,
  if not impossible to repurpose it.

* escape - A variable is said to "escape its scope" if it is defined in a
  particular scope and is "returned out" of the scope, either directly, or
  as part of another variable that escapes.

  If you've written Haskell, you might've encountered a
  ["type variable would escape its scope"](https://stackoverflow.com/q/27247620/2682729)
  error at some point -- this is a similar idea. The binding is introduced
  by a `forall`, and the thing that is "escaping" is the identity of the
  type variable. This mostly happens if you're mucking around with higher-rank
  types, because you're working with "smaller scopes" in a way, introducing
  type variables like plain variables (in a tree-like fashion) instead of at
  the "head" like you'd normally would when limited to rank-1 types.

  Before today, I didn't really have a solid intuition for the use of the word
  "escape" here.

  Closely related is a compiler pass called "escape analysis", which is
  often used by garbage collected languages to reduce heap allocation.

Turns out, many of these are used in physics in similar contexts:

* bound state - A state that tends to remain localized. For example, you
  could put a macroscopic object in an actually well, or a microscopic object in
  an infinite square well potential :stuck_out_tongue:. Even finite wells
  often have bound states.

* free state - A state that is not localized. (Yes, I know there are quasi-bound
  states but let's not go there...)

* potential well - A well, but potentially more abstract :stuck_out_tongue:.
  For example, the sun creates a gravitational potential well. This is kinda'
  like a scope.

* escaping trajectory - I guess this is self-explanatory. Could be in physical
  space (like the Voyager spacecrafts) or more abstract.

* bound trajectory - A trajectory that doesn't escape.

* cavity (optics) - Cavities are used to confine electromagnetic waves to a
  particular region in space. Maybe we should rename scopes to cavities?
  Or rename cavities to scopes? :thinking:

* tunneling - A particle can "escape" a potential well by tunneling. Calculating
  escape probability as a function of time is often valuable.

Is it just a coincidence due to etymology/English meanings of these words?
That does seems like the most plausible hypothesis. Nonetheless, it is
interesting to me that there are many similar terms between two fields that
I have some experience in :smile:, given that the fields are don't seem obviously
connected, unlike say programming languages and linguistics.
