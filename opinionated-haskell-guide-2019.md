# An opinionated beginner's guide to Haskell in mid 2019

This is mostly intended as a guide for people who are beginners to Haskell,
or have experience in other similar languages and are looking to learn Haskell.
Depending on where you are in your Haskell journey, parts of this guide might
not make sense. That's perfectly normal, relax.

(This is also not for you if you're coming to Haskell because you're interested
in the connections with type theory or category theory. I'm the wrong person for
that.)

As a quick litmus test, if you've written more than 10k lines of Haskell, this
guide is probably not for you. Also, I do not have the time/energy to
justify each and every statement. Please cut me some slack.

This is not a style guide. If you want one, you could use
[Kowainik's style guide](https://kowainik.github.io/posts/2019-02-06-style-guide)
or [Johan Tibbe's style guide](https://github.com/tibbe/haskell-style-guide/blob/master/haskell-style.md).
I'm sure there are more, but these are good. Overall, I don't think
"style guides" are a big deal in Haskell land (compared to say something
like PEP 8 in Python, or standard formatters like `gofmt`/`elm-format`).

This guide is, very loosely speaking, an extension of Alexis King's excellent
[An opinionated guide to Haskell in 2018](https://lexi-lambda.github.io/blog/2018/02/10/an-opinionated-guide-to-haskell-in-2018/)
 (henceforth OGH18). As such, I will not try to repeat the points made there,
unless I have a different opinion. I will point you to the relevant portions of
that guide and expect that you will read them.

Enough chit chat and disclaimers. Let's go.

## Key principle

Make decisions based on popularity and simplicity.

Once you're comfortable, feel free to go
[kamikaze](https://en.wiktionary.org/wiki/kamikaze) with your library
and language extension choices.

Also, feel free to retreat once you get too deep. There is no shame in enforcing
fewer invariants using types if that means you can understand your code much
better.

## Learning

### How do I learn Haskell?

Different people learn differently. Without knowing more about you, it is hard
to make a blanket one-size-fits-all recommendation.

If you're new to functional programming, the
[CIS 194 Spring 2013](https://www.seas.upenn.edu/~cis194/spring13/) course
is widely considered as a great resource, particularly if you're new to
functional programming.

If you already have some familiarity with functional programming, it might be
best to have a small project in mind and work towards it.

[Haskell Programming from First Principles](http://haskellbook.com/)
(a.k.a. haskellbook) is also often recommended for beginners. It doesn't
assume previous programming experience. I've found it useful to some extent,
if somewhat slow/ponderous.

I found [Real World Haskell](http://book.realworldhaskell.org/read/)
to be more suited to my learning style. At this point, some parts of the book
are a bit outdated (some libraries have been superseded by better alternatives),
but most of the core is still solid.

There's also [Programming in Haskell](http://www.cs.nott.ac.uk/~pszgmh/pih.html).
I can't really comment on it as I haven't read it, but I've heard good things
about it. The good thing is that it has many exercises, and it is written by
Graham Hutton, who has a lot of experience teaching Haskell.

[Learn you a Haskell for great good!](http://learnyouahaskell.com/) is somewhat
fun/whimsical if you're getting started, but I don't find it very practical.
However, it lacks exercises, which means that it isn't well-suited as a primary
learning resource. It also has some questionable choices of examples.

I highly recommend Oskar Wickstrom's excellent
[Haskell at Work](https://www.youtube.com/channel/UCUgxpaK7ySR-z6AXA5-uDuw/videos)
YouTube channel.

The key thing, in my opinion, is to practice. Whether it is through doing
a project, or following a book and doing exercises, or something else, there's
no real substitute for actually writing Haskell code and facing the type
errors head on.

### Getting help

If you're having difficulty understanding things, please ask! You getting stuck
means that you're less likely to stick around, and that makes me sad because I
want more people to enjoy learning and using Haskell.

When asking (and specifically for recommendations), it is important that
you tell the opposite person about your experience level, so they can adjust
their recommendation(s) accordingly.

If you don't have much experience asking programming-related questions online,
read a how-to about it (e.g. this [one](https://www.propublica.org/nerds/how-to-ask-programming-questions)
is short and sweet), which will help others help you better.

There are several places to ask questions. The [r/haskell](https://www.reddit.com/r/haskell/)
subreddit has several links in the sidebar, which you may find useful. If you
find general functional programming chat groups (e.g. on Slack or Discord), it
is likely that someone there might be able to help you with Haskell-related questions.
For example, you could try the [FPChat Slack channel #haskell-beginners](https://functionalprogramming.slack.com/messages/C04641JCU/)
([invite](https://fpchat-invite.herokuapp.com/)).

Twitter is also an option (although the medium makes it harder to communicate IMO),
use the `#haskell` tag.

My personal preference was to use [r/haskellquestions](https://www.reddit.com/r/haskell/)
and read answers on StackOverflow, but I recognize that may not work for everyone.

## Setup

### Operating system

Usually, you do not have flexibility in what operating system you can use.
This section only applies in case you do have that flexibility.

In my experience, most Haskellers use Linux, followed by macOS, followed by
Windows. While Haskell certainly does work fine (with some caveats) on all three
platforms, if it possible for you to use the more popular platforms and you
are comfortable doing so, you should do so, as this makes it easier for more
people to help you.

### Build tools

For any non-trivial project, you're going to need a build tool, instead of
running the compiler by hand.

I recommend using `stack` or `cabal v2-build` (also called "new-build" or
"Nix-style" builds). I use `stack` most of the time, but I've used
`cabal v2-build` on occasion and it works fine.

If you bump across instructions to use `cabal install foo` or
`cabal sandbox foo`, stop and retreat. That advice is most likely outdated,
don't follow it.

Some people use the `nix` package manager. I recommend not using `nix` when
you're starting out because it is relatively complex and has less
beginner-friendly documentation compared to `stack`.

OGH18's "Build tools and ..." section provides a good mental model for `stack`.
This is super important! Read it.

### Editor support

There are varying levels of support for Haskell in different editors via plugins.
They're based on different projects such as `intero`, HIE (Haskell IDE engine),
`dante` and more.

In my experience, tools/plugins based on `intero` seem to work the most
reliably. I've had success with using `intero` based plugins in Spacemacs
and VS Code.

To be clear, I'm not saying "just use Intero". If you're able to get other
things to work for you, great! All I'm saying is, do try an `intero` based
plugin if all else fails for you.

### Other tools

#### Hoogle

[`hoogle`](https://hoogle.haskell.org/) is a Haskell-specific search engine
where you can not only search for functions/modules/packages by name, but also
by type (well, at least for functions anyways).
From my POV, Hoogle is invaluable in the process of learning to think with types.
I encourage using it on a regular basis.

Hoogle can also be set up to work locally (see the [Readme](https://github.com/ndmitchell/hoogle)
for details). In case you don't want to do that, you might find it helpful to
set up a browser shortcut to use online Hoogle.

In Firefox specifically, these are called "custom search engines". If you set up
Firefox sync, you can use the same shortcut with Firefox on your phone, so
that you can quickly hoogle things while you're sitting on the toilet.
You're welcome.

#### Hlint

[`hlint`](https://github.com/ndmitchell/hlint) is an excellent linter that will
suggest improvements to your code. In some languages, you might have had a bad
experience with linters being excessively naggy. `hlint` isn't like that.
It's kinda' like having a senior Haskeller sitting by your side helping you out.
Use it!

#### Online compiler

Sometimes you want to share code with a friend or someone else to show what
you're trying to do/getting stuck at.

You can use [repl.it](https://repl.it/languages/haskell) as an online compiler
for this, making it easier for others to help you. It allows you to run code
as well.

[I'm sure other similar services exist, this is just one example.]

#### Setting up CI

Setting up CI can be frustrating sometimes.
Like with a lot of programming problems, it can be solved by copy-and-paste.

If you're using `stack` as your build tool, the documentation describes how
to setup Travis CI [here](https://docs.haskellstack.org/en/stable/travis_ci/).

You could also get the CI configuration from another Haskell library (e.g. if
you're using `cabal`). For example,
[here](https://github.com/kowainik/relude/blob/master/.travis.yml) is
the Travis CI file for the `relude` library, which has configuration for both
`stack` and `cabal`. Or pick some other library, doesn't matter.

If you want something more configurable and cross-platform, take a look at
[packcheck](https://github.com/composewell/packcheck).

If you're using Gitlab to host your source code, Gitlab CI is also a solid
option. (TODO: Add a link on how to setup Gitlab CI easily for a Haskell
project.)

## Libraries

### Which library do I use for X?

Consult the [Aelve guide](https://guide.aelve.com/haskell) and
[State of the Haskell ecosystem](https://github.com/Gabriel439/post-rfc/blob/master/sotu.md)
for recommendations.

Pick a library that is **well documented**. Your future self will thank you.

Do not rely on search hits for old (read: 5+ years) blog posts or old StackOverflow
answers.

If still in doubt, ask.

### Don't use a custom Prelude

Prelude is the module that is automatically imported in every Haskell module.
Unlike some other languages, Haskell has many different custom Preludes, written
by different groups of people, which have different trade-offs.

Stick to the default Prelude that comes with the `base` standard library.
Don't use a custom Prelude while you're a beginner.

## Your code

### Application architecture

Depending on your experience in other languages, the application architecture
you might be comfortable with may not work well in Haskell.

Also, it's almost impossible to provide general advice here.

Ask others for a sanity-check or help for your specific problem.

### Pure functions

Are the best thing since sliced bread.

This is not a Haskell-specific idea. In other communities, it is often phrased as
"functional core, imperative shell".

Learning how to make most of your code rely mostly on pure functions is
kinda' important. As a beginner, it might be hard to avoid making most of your
code pure (say 80%+). That's ok. Ask people for tips on how to do so. If the
solutions are understandable (i.e. don't seem too complicated), try them out.
If the suggested solutions seem too complicated, leave them for a bit and try
them out later when you're more comfortable.

### Partial functions

Don't write them. Your future self will be thankful.

Don't use them. Most commonly, this comes up when working with lists, as some
of the functions in `base` do not have sufficiently precise types. Trying using
`NonEmpty` from `Data.List.NonEmpty` if you can.

If you really must, use a partial function but write a short justification in
a comment for why there won't be any exceptions thrown. Sometimes, trying to
write that justification will uncover a bug in your reasoning.

IO related functions are somewhat notorious for throwing exceptions for
relatively common errors, such as a missing file. Try to be extra careful and
think about possible failure modes that you want to address when working with
functions in IO.

### Using records

Haskell's built-in records work just fine for small to medium applications.
If you're having problems with duplicate field names (or constructor names),
add a prefix corresponding to the data type to please the compiler.
Do not use the `DuplicateRecordFields` extension.
It often creates more problems than it solves, and by that time it might be too late.

You will see the `lens` library recommended in several places. You may even hear
extreme takes like "programming without lens is like having your hands tied behind
you back". Ignore, define some helper functions (very often, this is good enough),
and move on.

If you *really* must use a lens library (to see what all the fuss is all about or
otherwise), I recommend using `microlens` (or `microlens-platform`). Read
this [lens tutorial](https://hackage.haskell.org/package/lens-tutorial-1.0.3/docs/Control-Lens-Tutorial.html),
it will cover a large fraction of your use cases.

(Note: The tutorial refers to the `lens` library, not `microlens`.
Roughly speaking, you can replace the `Control.Lens` in the tutorial with
`Lens.Micro` and the code will still work.)

### Error handling

#### For libraries

Define custom error ADTs as appropriate. Sometimes, these may be for one
specific function, or they may be shared for a few specific function.
Even if it just used by one function, please define it separately!
Try to avoid [boolean blindness](https://www.cs.cmu.edu/~15150/previous-semesters/2012-spring/resources/lectures/09.pdf)
(sometimes also manifests as `Maybe` blindness).

Return `Either MyError [..]` from the function that might fail.

Usually, there is no good reason to deviate from this. One such good
reason is that it is super obvious that there is only reason for failure
(and it will continue to be like that), in which case, using `Maybe` is fine.

#### For applications

The error handling strategy will depend on overall application architecture.

When in doubt, default to the simple strategy of using `Either` as documented
above. If still concerned, ask someone.

One difference from libraries it is more likely that you want to aggregate errors,
instead of exiting at the first error. In such a situation, you can either use
`Either [MyError]` (simple and no additional dependencies, but doesn't have
the semantics of "aggregate errors")
or use the [`Validation`](https://www.stackage.org/haddock/lts-13.25/validation-1/Data-Validation.html#t:Validation)
data type (that library has a heavy dependency footprint).

Or you can roll your own data type similar to `Validation` and avoid the
dependency. Don't forget to give proper attribution.

### Testing

[`hspec`](http://hspec.github.io/) is a good option for writing and organizing
tests.

If you're interested in trying out property-based testing, I recommend
using [`hedgehog`](https://github.com/hedgehogqa/haskell-hedgehog).
[`Quickcheck`](https://hackage.haskell.org/package/QuickCheck) is more well-known
I think, and it is also a fine option.

You can write documentation tests too using [`doctest`](https://github.com/sol/doctest)

### Abstraction (alt. but ma', I wanna' be one of the cool kids!)

Try not to over-abstract. When writing Haskell, it is *extremely* tempting to
build more sophisticated abstractions, to the detriment of everything else.

![Thanos and Gamora meme](https://i.imgflip.com/33mzg6.jpg)

On reddit, or mailing lists, or on Twitter, or elsewhere, you'll often find
people talking about things that sound really cool. For example, recently,
there is a lot of hype around dependent(-ish) types and algebraic effects.
To be honest, I don't really understand all the tradeoffs around these myself,
and I've been writing Haskell on and off for about 18 months now.

People don't talk all that much about code that uses simple patterns,
because it is "boring". In my view, your first task as a beginner is to learn
to write "boring" code well.

The best code is the code that doesn't exist.
The next best code is the one that compiles without type errors for reasons
that you understand.

## Debugging

### Print debugging

You might think "oh no, I can't print debug in Haskell inside pure functions :frowning:".
You're wrong (kinda')! Use the [Debug.Trace](https://hackage.haskell.org/package/base-4.12.0.0/docs/Debug-Trace.html)
module for quick print debugging. Once you're done debugging, delete the import.
My "trick" for this is to do `import qualified Debug.Trace as D` so that
the compiler warns me about unused imports once I remove all calls to `trace`
in the module.

### Using a proper debugger

ghci [has a debugger](https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/ghci.html#the-ghci-debugger).

I haven't really used it in practice, so I can't really comment on the
experience. Sorry.

### Profiling

GHC optimizes code fairly well, so normally you shouldn't be facing major
problems with execution speed (throughput) for a beginner/intermediate
project. However, in case you do, threading timing everywhere is kinda'
tedious and annoying (because you want the core of the code to be pure).

You can use GHC's built-in instrumentation profiling. Long story short,
it will generate a report (either human readable or in JSON for consumption
by other tools) that shows how much time different functions take to run.

For simple cases, reading the profile manually is sufficient. For more complex
situations, I've used
[flamegraph](https://github.com/brendangregg/FlameGraph) and highly recommend it.
You can use `ghc-prof-aeson-flamegraph` as glue to convert GHC's
JSON-formatted profile to something suitable for `flamegraph`.

```
# Make sure that ghc-prof-aeson-flamegraph is on your $PATH.
#
# stack install ghc-prof-aeson-flamegraph

# git clone https://github.com/brendangregg/FlameGraph.git
# cd myproject

# The work-dir option prevents stack from overwriting your default build.
# Otherwise, rebuilding with/without profiling will overwrite your builds,
# meaning that you need to recompile a LOT of code again and again.
#                         ↓
stack build --profile --work-dir=".profile-dir"
#           ^-- instrument the binary for profiling
#               this can result in a 3~10x slowdown

#        Ask the RTS to record a profile using the json (j) format ↓
stack exec --work-dir=".profile-dir" mybinary -- myinput.csv +RTS -pj

cat mybinary.prof | ghc-prof-aeson-flamegraph | ../FlameGraph/flamegraph.pl > graph.svg
# open graph.svg in a browser
```

Reddit user gilmi [recommends](https://www.reddit.com/r/haskell/comments/c1srzr/an_opinionated_beginners_guide_to_haskell_in_mid/erfc7x0)
[profiteur](https://github.com/jaspervdj/profiteur)
for exploring profiles. The installation instructions there suggest using
`cabal install profiteur`. As I said earlier, don't use `cabal install`.
Instead, you can install `profiteur` as follows:

(TODO: Add instructions or link on how to setup `profiteur`.)

This will avoid mutating the global state of packages, preventing problems in the future.

## I'm still concerned ...

### why are there so many language extensions? (alt. why is there no standard Haskell)

It's a matter of experimentation. Haskell is both a research language and an
industrial language. Don't worry about this. It's mostly not gonna' affect your
day-to-day work.

I'm mostly in agreement with the list of 34 default extensions in OGH18.
That doesn't mean you *need* those extensions. You probably don't need most of them
as a beginner. It means that those extensions are relatively harmless.

Sometimes, GHC will suggest enabling a language extension because it thinks you
want a particular language feature based on some code you wrote. Nothing bad's gonna
happen, trust me. Just enable it. Worst thing that might happen is you get a confusing
type error. Most likely, your code is gonna' compile and everything's fine and rosy.
One exception to this rule (in my experience) is `AllowAmbiguousTypes`.
Sometimes when GHC suggests using it, there's a bug in your code and you should
specify the types of intermediate computations.

There will be a point at which you will run into the monomorphism restriction.
It might sound scary, but it is mostly annoying, not scary.
You will land at this [question](https://stackoverflow.com/q/32496864/2682729) on StackOverflow.
You will read the answers and your problem will be solved. It is known.

## Closing thoughts

When in doubt, please ask for help. You miss 100% of the thunks you don't force
(try not to over think it, this analogy doesn't really work).

Happy Haskelling.

## Thanks

Thanks to reddit user TracyChavez for [this comment](https://www.reddit.com/r/haskell/comments/c141zb/any_movement_on_development_of_haskell_the/erdl5f6)
which prompted me to write this post.
