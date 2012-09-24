---
layout: post
title: "Small and Fast"
description: "How to write small programs that run fast"
category: programming languages
tags: [programming languages]
---
{% include JB/setup %}
# SMALL AND FAST

My favorite article comparing programming languages is [The speed, size and dependability of programming languages](http://blog.gmarceau.qc.ca/2009/05/speed-size-and-dependability-of.html) by Guillaume Marceau.  Guillaume plots program size against execution time for programs written in various languages, implementing benchmarks from [The Computer Language Benchmark Game](http://shootout.alioth.debian.org/).  With the X axis program size and the Y axis execution time, the ideal language would have all of its programs plotted in the lower left corner of the chart.  Evidently, his idea was popular enough that now the Benchmarks Game keeps an [up to date version](http://shootout.alioth.debian.org/u64/code-used-time-used-shapes.php) of these charts.

Programming language features besides program size and execution speed might be more important in some contexts, but broadly speaking, much of the history of programming language design is a search for the lower left hand corner of Guillaume's chart.  Paul Graham argures for a programming language ["succinctness = power."](http://www.paulgraham.com/power.html)  Faster programs are more responsive, answer questions faster, and consume fewer processing resources.  Writing short programs is often at odds with writing fast programs, so achieving both simultaneously is a kind of Holy Grail of programming language design.

This article presents a high level, qualitative review of strategies for writing small and fast programs.  To keep the scope manageable, only performance in a single thread and address space is considered, ignoring parallel and distributed computing concerns.  Those are very important aspects of writing small and fast programs, but not the topic of this article.

Even with these caveats and constraints, I am sure I have missed a lot and gotten a lot wrong.  I hope this spawns a dialog better informing me how to write small and fast programs.

## LANGUAGE STACKS

If it is difficult to design a single language yielding small, fast programs, an alternative is to write your program in multiple languages.  Many languages implement a Foreign Function Interface (FFI) for exactly this reason.  The language calling the FFI is usually optimized for writing small programs, and the foreign language is usually optimized for writing fast programs, often a dialect of C.

An example is the [NumPy/SciPy](http://numpy.scipy.org) environment, a popular tool for scientists, engineers, statisticians and others to quickly crunch large amounts of numeric data and visualize the results.  These users are often not professional programmers, and the work is often exploratory,  so the ability to quickly write short programs, immediately see the results, and rapidly iterate are critically important.

Python fulfills these requirements in NumPy.  Python supports a Read Eval Print Loop for executing code a line at at time and immediately displaying the results, as seen [here](http://www.scipy.org/Cookbook/OptimizationDemo1).  Python is also good for writing short programs.  It doesn't require explicit type declarations for variables or functions and includes features like [list comprehensions](http://thejosephturner.com/blog/2011/01/07/project-euler-problem-4/) for concisely filtering and transforming lists, among many other features to support writing short programs.

However, Python is not well suited for rapidly crunching large amounts of numeric data.  Those tasks are performed in NumPy by Fortran libraries like [LAPACK](http://www.netlib.org/lapack/) and other libraries written in C.  Fortran and C require programmers to provide explicit information about the types and sizes of values and variables, which Fortran and C compilers utilize to emit highly optimized executable code.  Of course, these same requirments result in longer programs compared to Python.

In Guillaume's chart, these language characteristics are reflected by C and Fortran's positions against the left hand side of the chart (fast but longer programs) and Python in the lower right (short but slow programs).

Key to producing small, fast programs using the "language stack" strategy is understanding which operations in the high level languages are passed off to efficient implementations in the lower level language.  For example, one semester I decided to implement all the homework assignments for a particular course in Python and NumPy.  The first few assignments went without a hitch, but for one assignment, I realized my program was not going to finish processing the assignment dataset before the assignment deadline.  In the NumPy source I found a function consuming much of the program execution time, implemented entirely in Python.  I didn't see any easy way to avoid calling that particular function, and didn't have enough time to figure out how to patch NumPy, so I re-implemented my program in Java using the [JAMA](http://math.nist.gov/javanumerics/jama/) library which completed in time for me to turn in my assignment only slightly late.

In general, knowing which data structures and functions have been optimized in languages like Python, Ruby and Perl can make a big difference in writing fast programs.  As Alex Rubinsteyn notes in his [performance tuning  tips](http://blog.explainmydata.com/2012/07/expensive-lessons-in-python-performance.html) for number crunching with NumPy and other Python libraries, "have faith in Python's core operations."

> On the other hand, once the obviously vectorizable stuff is off-loaded to library code, then I'm often surprised just how efficient Python's operations can be. I can build lists of startling length, index into dictionaries all day long, construct millions of simple objects, and it all adds up to a small fraction of my total runtime. 

This is because Python's core operations are implemented in highly optimized C, and thus much faster than code written entirely in Python.

There are many other examples of the "language stack" strategy.  Many game engines are written in C++ and scripted with [Lua](http://stackoverflow.com/questions/5053134/what-is-a-good-game-engine-that-uses-lua).  Objective C and [MacRuby](http://macruby.org/)/[RubyMotion](http://www.rubymotion.com/) are an interesting language stack.  Objective C adds SmallTalk style message passing as an extension to C, making it a nice semantic fit with Ruby, which also draws inspiration from SmallTalk style message passing.

With the availability of higher level languages like Jython, the language stack strategy can also be deployed on the Java Virtual Machine.  In Guillaume's chart Java is closer to C and Fortran than Python.  Where I work, our core libraries are implemented in Java but I do a lot of scripting and exploratory development invoking those libraries from Jython.  The [Incanter](http://incanter.org/) project is similar to NumPy, with Java playing the role of C/Fortran and Clojure playing the role of Python.

Tools like [Python Weave](https://github.com/scipy/scipy/blob/master/scipy/weave/doc/tutorial.txt) take the language stack strategy even farther by allowing programmers to include C code directly in Python source files.

## OPTIONAL TYPES

Picking one language requiring explicit type information to help the compiler emit fast code, and one language not requiring explicit type information to enable smaller programs, gives a lot of flexibility in trading off program size and execution time.  However, switching between languages and toolsets then integrating the results to get a working system is less productive than writing everything in a single language, everything else being equal.

So why not start with concise, expressive code cleanly expressing a program's intent, then add type declarations to improve performance bottlenecks and hotspots?  Common Lisp allows precisely this approach, as described in [Common Lisp the Language](http://www.cs.cmu.edu/Groups/AI/html/cltl/clm/node103.html).

>Declarations allow you to specify extra information about your program to the Lisp system. ...[D]eclarations are completely optional and correct declarations do not affect the meaning of a correct program.

> ...

> [D]eclarations are of an advisory nature, and may be used by the Lisp system to aid the programmer by performing extra error checking or producing more efficient compiled code. Declarations are also a good way to add documentation to a program.

In other words, write a correct, straightforward implementation of the program, then add declarations to get the required performance characteristics, among other benefits.


Didier Verna documents applying this technique to make Common Lisp programs [as fast as equivalent C programs](http://www.iaeng.org/IJCS/issues_v32/issue_4/IJCS_32_4_19.pdf).  For his experiments, Didier "benchmarked, in both languages, 4 simple [image processing] algorithms to measure the performances of the fundamental low-level operations: massive pixel access and arithmetic processing."

Here is a sample C function from his experiments.

    void add (image *to, image *from, float val) {
        int i;
        const int n = ima->n;
    	for (i = 0; i < n; ++i)
            to->data[i] = from->data[i] + val;
    }

Here is the corresponding, unoptimized version of that function in Common Lisp.

    (defun add (to from val)
        (let (( size (array-dimension to 0)))
            (dotimes (i size)
                (setf (aref to i) (+ (aref from i) val)))))

Running the interpreted version of this function in CMU CL was 2300 times slower than the C version, the compiled, unoptimized version was 60 times slower, and the compiled, optimized version was 20 times slower.

Here is the same function with type declarations.

    (defun add (to from val)
        (declare (type (simple-array single-float (*)) to from))
	    (declare (type single-float val))
	    (let (( size (array-dimension to 0)))
	        (dotimes (i size)
	            (setf (aref to i) (+ (aref from i) val)))))

With declarations like these added, the Lisp code compiled by CMU CL with fully optimized, unsafe settings performed almost identically to C for most of the algorithms.  CMU CL was 3 times slower at integer division but 10 percent faster at float division.

Why does adding type declarations make the program so much faster?  Didier gives a few reasons.

> ...[C]ontrary to C, variables and function arguments ... could hold any Lisp object.  As a consequence, the compiled Lisp code has to check dynamically that the variables we use are of the proper type with respect to the operations we want to apply to them.

> ...

> Lisp objects contain type information plus a pointer to the real value. And obviously, pointer (de)referencing is causing a major performance loss.

One caveat to this approach is that different Common Lisp "[c]ompilers may behave very differently with respect to type declarations and may provide type inference systems of various quality."  Didier presents an example where CMU CL performs optimizations that Allegro CL does not.  In general, the capabilities of a particular Common Lisp implementation determine what declarations are needed to meet performance requirements, if possible at all.

Michael Weber brings to light another caveat to this approach.  He implemented the [reverse-complement benchmark](http://www.foldr.org/~michaelw/log/programming/lisp/reverse-complement-benchmark) from the Computer Language Benchmarks game in Steel Bank Common Lisp (SBCL), beating every other implementation except gcc.  Note this is from 2006, and the current SBCL implementation is further down the leaderboard for that benchmark.  However, Michael also points out "Approaching the speed of C seems to force us to more or less write C in Lisp."  In other words, once enough declarations are added to give the Common Lisp compiler as much information as the C compiler, the Lisp code can be just as verbose as C code, if not moreso.  There is still the benefit of writing the entire program in one language, and declarations can be confined to performance hotspots, but C like performance does not come for free.

[Numba and Cython](http://jakevdp.github.com/blog/2012/08/24/numba-vs-cython/) make it possible to add declarations to speed up Python programs.  The article shows an example of a 1,000x speed up of a Python program with a couple of declarations for the Numba Just-In-Time compiler.  The same article shows an additional 30% improvement over Numba for a version with declarations for Cython, which "allows python and/or python-like code to be compiled into C for fast operations."

Clojure allows programmers to add [type hints](http://clojure.org/java_interop?f=print#Java%20Interop-Type%20Hints) to avoid reflection when invoking Java methods in performance critical parts of the code.  The example given in the documentation for this feature shows a factor of 10 speed up in a simple function by introducing a type hint.

Google's Dart language has optional static type declarations, but currently they are only used for tooling and compiler warnings and otherwise have ["no effect and no cost."](http://gotocon.com/dl/goto-aarhus-2011/slides/GiladBracha_and_LarsBak_OpeningKeynoteDartANewProgrammingLanguageForStructuredWebProgramming.pdf)

## TYPES WITH LESS TYPING

Providing optional, explicit type information in performance critical code regions allows programmers to trade off program size and execution speed without having to jump between languages.  What if we could give the compiler complete information about all the types throughout our program, while only having to type out those types explicitly in a very limited number of places?  Program size would benefit from eliminating most of the explicit type information, and compilers would have enough information to perform lots of performance optimizations.

This is the approach taken by functional languages like Haskell and OCaml.  In [OCaml for the Masses](http://queue.acm.org/detail.cfm?id=2038036), Yaron Minsky cites concision and performance among the reasons [Jane Street](http://www.janestreet.com/technology/ocaml.php) has built much of their infrastructure in OCaml.

> Our experience with OCaml on the research side convinced us that we could build smaller, simpler, easier-to-understand systems in OCaml than we could in languages such as Java or C#. For an organization that valued readability, this was a huge win.

> ...

> We found that OCaml's performance was on par with or better than Java's, and within spitting distance of languages such as C or C++.

Yaron credits type inference with making OCaml programs faster and more concise.

> One advantage OCaml brings to the table is type inference, which obviates the need for many type declarations. This leaves you with code that is roughly as compact as code written in dynamic languages such as Python and Ruby. At the same time, you get the performance and correctness benefits of static types.

As an example, Yaron presents a function that applies the function f to all three elements of a tuple.  Here is the OCaml implementation.

    let map f (x,y,z) =
        (f x, f y, f z)

Here is the equivalent function implemented in C#.

    Tuple<U,U,U> Map<T,U>(Func <T,U> f, Tuple<T,T,T> t) {
        return new Tuple<U,U,U>(f(t.item1), f(t.item2), f(t.item3));
    }

The OCaml code is much shorter but just as strongly typed as the C# code, without all of the generic type parameters.

The book [Real World Haskell](http://book.realworldhaskell.org/read/why-functional-programming-why-haskell.html#id528487) makes similar claims about the benefits of type inference in Haskell.  Type inference makes Haskell programs smaller than languages requiring explicit types.

> Although powerful, Haskell's type system is often also unobtrusive. If we omit explicit type information, a Haskell compiler will automatically infer the type of an expression or function. Compared to traditional static languages, to which we must spoon-feed large amounts of type information, the combination of power and inference in Haskell's type system significantly reduces the clutter and redundancy of our code.

A [study](http://haskell.cs.yale.edu/?post_type=publication&p=366) comparing Haskell productivity against several other languages lends support to the claim that Haskell yields concise programs.

> The results indicate that the Haskell prototype took significantly less time to develop and was considerably more concise and easier to understand than the corresponding prototypes written in several different imperative languages, including Ada and C++.

And type inference makes Haskell programs faster than "dynamic" languages that don't require explicit types.

> When we consider runtime performance, Haskell almost always has a huge advantage. Code compiled by the Glasgow Haskell Compiler (GHC) is typically between 20 and 60 times faster than code run through a dynamic language's interpreter.

One important difference, however, is Haskell's default "lazy" evaluation strategy versus OCaml's default "strict" evaluation strategy.  Haskell "defer(s) every computation until its result is actually needed."  On the one hand, this can avoid unneccessary computation and allows for the creation of "infinite" data structures (because only the data actually needed for the end result will be computed).  On the other hand, laziness makes performance very hard to predict.

Enforcing strict evaluation is often cited as a way to improve Haskell performance.  Johan Tibell presents several examples of improving performance by avoiding lazy evaluation in his [High-Performance Haskell](http://www.slideshare.net/tibbe/highperformance-haskell) slide deck.  The Haskell wiki has a section on improving performance through [strictness analysis.](http://www.haskell.org/haskellwiki/Performance/Strictness#Strictness_analysis)  As an aside, [this](http://blog.thezerobit.com/2012/07/21/immutable-persistent-data-structures-in-common-lisp.html) article also cites laziness as a reason for Haskell underperforming Standard ML (a language in the ML family, like OCaml).

> Every implementation of Standard ML (not lazy) I've tried has varied from just being somewhat to several times faster than Haskell where all computation is lazy. I think setting up all that delayed computation is expensive, and the only way I found to make Haskell perform within the same ball park as Standard ML, was to add hints here and there to get computations to go ahead and happen instead of building a stack of thunks.

On the other hand, Yaron praises the predictability of OCaml performance in [Caml Trading: Experiences in Functional Programming on Wall Street](http://www.haskell.org/wikiupload/0/03/TMR-Issue7.pdf).

> Another important aspect of OCaml's performance is its predictability. OCaml has a reasonably simple execution model, making it easy to look at a piece of OCaml code and understand roughly how much space it is going to use and how fast it is going to run.

Which is very important for Jane Street, as they need to perform [hundreds of thousands of transactions per second, with sub second latency](http://janestreet.com/yaron_minsky-cufp_2006.pdf).

Similar to the notion of "writing C in Lisp," writing highly performant code in high level functional languages can also require writing code that is less succint.  Yaron Minsky describes how this manifests in [OCaml.](http://www.haskell.org/wikiupload/0/03/TMR-Issue7.pdf)

> OCaml's approach to compilation has a cost, though. Programmers learn to write in a style that pleases the compiler, rather than using more readable or more clearly correct code. To get better performance they may duplicate code, expose type information, manually pack data structures, or avoid the use of higher-order functions, polymorphic types, or functors.

Neil Mitchell makes [similar comments](http://neilmitchell.blogspot.com/2008/05/haskell-and-performance.html) regarding Haskell.

> If you use GHC, with unboxed operations, written in a low-level style, you can obtain similar performance to C. The Haskell won't be as nice as it was before, but will still probably express fewer details than the C code.

And the Haskell performance wiki page [concedes](http://www.haskell.org/haskellwiki/Performance)

> The main caveat is that you may have to modify your code significantly in order to improve its performance.

Don Stewart [disagrees](http://donsbot.wordpress.com/2008/06/04/haskell-as-fast-as-c-working-at-a-high-altitude-for-low-level-performance/), claiming that it is possible to get "low level performance" while working at "a higher abstraction level."  However, while I didn't understand everything in the article, it seems the program he ends up with is not the most straightforward or succint implementation of the underlying algorithm.

In addition to optional type declarations, Common Lisp implementations can [infer](http://www.iaeng.org/IJCS/issues_v32/issue_4/IJCS_32_4_19.pdf) some of the types that are not explicitly specified.

> When not all types are provided by the programmer, modern Lisp compilers try to infer the missing ones from the available information.

For example, [here](http://common-lisp.net/project/cmucl/doc/cmu-user/compiler-hint.html#toc154) are the kinds of type inference performed by CMU CL.  Go infers types in [initialization statements](http://tour.golang.org/#12), and can infer whether an object [implements an interface without an explicit declaration saying so](http://tour.golang.org/#54).

## SUFFICIENTLY SMART COMPILER

Following our progression, what if we didn't specify types anywhere, and the compiler could still analyze our program, deduce all of the appropriate types, and apply the appropriate performance optimizations?  Half jokingly, this is sometimes referred to as the ["sufficiently smart compiler."](http://prog21.dadgum.com/40.html)

> If you're not familiar, here's a classic context for using "sufficiently smart compiler." Language X is much slower than C, but that's because floating point values are boxed and there's a garbage collection system. But...and here it comes...given a sufficiently smart compiler those values could be kept in registers and memory allocation patterns could be analyzed and reduced to static allocation. Of course that's quite a loaded phrase, right up there with "left as an exercise for the reader."

Perhaps the closest real world approximations to the Sufficiently Smart Compiler are Javascript implementations in major web browsers.  Since Javascript has a central role in web application development, improving Javascript performance is a necessity for improving client side web application performance.   That is why so much work has gone into improving performance in Javascript engines like Chrome's [V8](http://code.google.com/p/v8/), Safari's [Nitro](http://trac.webkit.org/wiki/JavaScriptCore), and Firefox's [SpiderMonkey](https://developer.mozilla.org/en-US/docs/SpiderMonkey).  However, since Javascript has almost no explicit type information, these engines use strategies to infer type information from running programs.

For example, V8 creates ["hidden classes"](https://developers.google.com/v8/design#prop_access) to replace dynamic lookups of object properties with fixed offset variable lookups.  In principle, any property with any value could be added to a Javascript object at any time.  In most real world Javascript programs, however, many objects are created with the same set of properties.  So when a new object is created or a property is set on an object, V8 will check to see whether a hidden class with the same name and properties already exists.  If not, V8 creates a new hidden class with the right set of properties to be reused the next time an object with the same set of properties is needed.

When I last checked the [Benchmark Game speed comparisons](http://shootout.alioth.debian.org/u32/which-programming-languages-are-fastest.php), Javascript V8 was slower than most of the explicitly typed and type inference languages like Haskell and OCaml, but much faster than other languages without explicit types like Python, Ruby and Perl.  At the same time V8 code size [compares favorably](http://shootout.alioth.debian.org/u64/which-language-is-best.php?gpp=on&ifc=on&java=on&sbcl=on&ghc=on&csharp=on&racket=on&v8=on&hipe=on&vw=on&php=on&python3=on&lua=on&perl=on&yarv=on&xfullcpu=0&xmem=0&xloc=1&nbody=1&fannkuchredux=1&meteor=1&fasta=1&fastaredux=1&spectralnorm=1&revcomp=1&mandelbrot=1&knucleotide=1&regexdna=1&pidigits=1&chameneosredux=1&threadring=1&binarytreesredux=1&binarytrees=1&calc=chart) with Python, Ruby and Perl.

## SUMMARY

Providing more information to the compiler makes programs longer, but also gives the compiler more ways to make programs run fast.  This tension seems to be behind many of the choices made in programming language design.  Using multiple languages, providing optional types, inferring types, and determining types implicitly at runtime are all strategies for addressing this tradeoff.

I'm curious to hear if I've missed any major strategies, and I'm curious to hear about ways to make programs both small and fast having nothing to do with types.  For example, how does garbage collection or Lisp style macros address the speed and size tradeoff?

Let's discuss further at ... 