# Program Analysis Glosarry

[I](https://nimrodpar.github.io/) created this glossary due to a recent program analysis exploration task where I found myself struggling to keep up with the jargon and all the (many) different kinds of analyses.

<h1 align="center">
  <a href='https://docs.google.com/presentation/d/1DkInjfdAT6BGIMI2xHavxGgLqrV-seBDvNx0gUgU-6c/edit?usp=sharing'> -->Slides<---</a>
</h1>

A transcribed version of the slides in Github Markdown format follows.

---

# Intro/Disclaimer

* The goal of this glossary is to convey the differences between different types of program analyses by example.
* The notions and examples shown in the slides are example-driven, and are not quite formal
* [I](https://nimrodpar.github.io/) have some knowledge in program analysis, but may contain errors or inaccuracies.
  * Please suggest corrections/additions via Github issues

**Some of the Sources:**
* https://www.seas.harvard.edu/courses/cs252/2011sp/slides/Lec02-Dataflow.pdf
* https://en.wikipedia.org/wiki/Data-flow_analysis
* https://stackoverflow.com/questions/13397180/what-exactly-does-context-mean-in-context-insensitive-analysis#:~:text=A%20context%2Dsensitive%20analysis%20is,target%20of%20a%20function%20call.&text=Context%2Dsensitivity%20also%20makes%20the%20analysis%20expensive.
* http://pages.cs.wisc.edu/~fischer/cs701.f14/popl95.pdf


# Basic Definitions
_These apply to (almost) all analyses._

## Definition: A Program Analysis

* A mapping from each location in the program to an abstract state describing some trait for program variables.
  * Example traits: Possible range of values (a.k.a interval), Which variables may still be used later in the program (a.k.a live variables), Which computed expressions are still valid (a.k.a available expressions), etc. 
  
<p align="center">
  <img src="https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/4.png" width='80%'/>
</p>
  
_Speaker Notes:_

For the sake of simplicity, we define a very basic notion of a program analysis. The example we show here and in the next 3 slides is an interval analysis, which simply tried to track the range of values for variables.

`x = nothing` means that x is not initialized so we don’t have any valid possible value for it. Another notation for this is `x = ⟂` (also called ‘bottom’).

Note: some analysis treat uninitialized variables as having *all possible values* (since this may be the case for some languages like C), which is usually denoted as `x = ⟙` (also called “top”).

<p align="center">
  <img src="https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/5.png" width='80%'/>
</p>
  
_Speaker Notes:_

The abstract states here does not seem abstract at all right? The values are very concrete and well defined. 

This is because of:
  1. the very simple program we are analysing 
  1. the “abstract domain” we are using. 

An abstract domain simply keeps track of certain properties for the (variables in) the program. In this example the properties are integer values, but can also be stuff like “is the variable null or not?” or “is the value of the variable odd or even”?. 

Abstract domains in program analysis have many rules that are required for correctness and are a very big and deep field of study. We won’t elaborate any further, just remember that their purpose is to keep track of (properties of) variable values. 

## Definition: The Join Operation ⨆

* An operation for...joining abstract states flowing from multiple paths.
* Also called a "May Analysis".

<p align="center">
  <img src="https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/6.png" width='50%'/>
</p>
  
_Speaker Notes:_

So, what do we do in the case of the pos() function? What is the abstract state at the `L3` location (i.e., what are all possible values for the variable `x`)?

Since we want to cover all possible values of variable `x` (this is also called “over-approximating”), we need to account for the values flowing into `L3`. The values are determined by the two branches of the if statement. So, we somehow need to account for the states that we have for `L1` and `L2`.

<p align="center">
  <img src="https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/7.png" width='70%'/>
</p>
  
_Speaker Notes:_

The operation for accounting for all states flowing into a program location is called a Join operation (denoted ⨆).

Basically the operation means “the value for `x` in `L3` can be either the value of `x` in `L1` or the value of `x` in `L2`”. 

Here, we join by simply ORing all the possible states flowing in. But this can be done differently. For instance, instead of 2 values for `x` in two sub-states, we can have `{ x = [0,42] }` i.e., `x` can be any value between 0 and 42. Since we are over-approximating, this result is okay but also very inaccurate. 

This sort of compromise (also called abstraction) is needed for efficiency. One of the major goals of program analysis is to create efficient and scalable analyses that are still accurate enough to be useful.

## Definition: The Meet Operation ⊓

* Merging abstract states flowing from multiple paths while keeping facts that are *true on all paths*.
* Also called a "Must Analysis".

<p align="center">
  <img src="https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/8.png" width='60%'/>
</p>

_Speaker Notes:_

For some analyses, we do not want to account for all possible states on all paths. 

That’s the case for the the available expressions analysis shown in the slide. This is an analysis used by compilers to determine what temporary computations can be re-used and need not be calculated.

In the example, the expressions `2*y` and `3*x` are used multiple times so it would be wise to re-use the calculation if possible. The analysis tracks the computed expressions by keeping tabs on the inputs to each expression to check if it had changed. 

For instance, `2*y` is computed in L0 and is therefore added to the state at the location (the state is the list of available expressions). At L1 location, `2*y` is no longer available since `y` changes and is removed from the state (but kept in `L2`).  `3*x` is added to the state at `L1` and `L2` since it was computed in the if condition and `x` does not change in any of the branches. 

Arriving at `L3`, to correctly know which expressions are available and need not be recomputed, we must only consider the expressions that are available in all paths leading to `L3`. This means we must meet the states from `L1` and `L2`, and keep only the expression shared by both-- `3*x`.

# Analyses Types

## May Analysis

* (as seen before) an over-approximating analysis that uses the join ⨆ operation to merge states flowing from multiple paths.

<p align="center">
  <img src="https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/7.png" width='70%'/>
</p>


## Must Analysis

* (as seen before) an under-approximating analysis that uses the meet ⊓ operation to merge states flowing from multiple paths.

<p align="center">
  <img src="https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/8.png" width='60%'/>
</p>
  
## Forward Analysis

* An analysis where the abstract state flows forward.
  * The state at each program point is derived from the states of the **preceding** program points.

  ![Forward Analysis](https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/12.png)
  
_Speaker Notes:_

You may not have noticed, but the way states at each location were computed is by using the states from previous lines and accounting for what happens at the current line (also called applying a “transformer”).

This sort of analysis, where you use the states for (direct) previous lines, is called a forward analysis. Most analysis are forward ones, including what we saw so far.

<p align="center">
  <img src="https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/13.png" width='85%'/>
</p>
  
_Speaker Notes:_

So that state from `L1` is taken as the initial state for `L2` (i.e., `in(L2) = L1`), and the operation `x=1` at `L2` is accounted for (the semantics of `L2` i.e., [| `x = 1` |]) and we get the resulting state for `L2` (i.e., `out(L2) = {x =1, y = nothing}`.

## Backward Analysis
* An analysis where the abstract state flows backwards.
  * The state at each program point is derived from the states of the **succeeding** program points.
* Example: Live Variables analysis.
  * An analysis used by compilers to determine which variables are no longer needed and can be freed.

<p align="center">
  <img src="https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/14.png" width='50%'/>
</p>
  
_Speaker Notes:_

This analysis starts the end of the program. It identifies that `L4` uses `y`, so `y` is live at `L4`.

<p align="center">
  <img src="https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/15.png" width='80%'/>
</p>
  
_Speaker Notes:_

To determine which variables are live (i.e., required and can’t be freed) at `L3`, the state from `L4` is taken, and the statement in `L3` is examined. The analysis identifies that `x` is needed at `L3`, so it is added to the state.

Note that we already know that `y` can be freed after `L3`.

<p align="center">
  <img src="https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/16.png" width='55%'/>
</p>

_Speaker Notes:_

The final result of the analysis.

## Flow-Sensitive Analysis

* An analysis that takes the order of instructions into account.

  ![Flow-Sensitive Analysis](https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/17.png)
  
_Speaker Notes:_

This (concise) slide shows the state for the exit point of f() (a.k.a. post-condition).

On the left we have a result for a flow-insensitive analysis where instruction ordering is ignored. In this sort of analysis, basically everything that happens everywhere in the program (w.r.t. Each variable) is tracked, and no overwriting is done. Therefore we get the state on the left tracking both assignments to `x`.

On the right we have a flow-sensitive analysis which tracks instruction ordering. The assignment of `x = 2` therefore overwrite the previous assignment in the state, resulting in the more precise `{x=2}` state at program exit.

Why would anyone use a flow-insensitive algorithm? They are simpler to specify, and can be faster. Therefore they are useful in some types of analysis where the precision gap may be small (e.g., https://www.cs.colorado.edu/~bec/papers/sas11-ptaprecision.pdf).

## Path-Sensitive Analysis

* An analysis that takes branch conditions into account.

  ![Path-Sensitive Analysis](https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/18.png)
  
_Speaker Notes:_

As before, the state of the left shows the result of a path-insensitive analysis at the exit point of `f()`. The `x > 0` branch condition is unaccounted for.

The path-sensitive state on the right shows the result at the exit point for `f()`, where path conditions, i.e., the branch condition and its negation, are taken into account. This result in a bigger and more informative state (but also more expensive).

## Context-Sensitive / Interprocedural Analysis

* An analysis that takes the calling context into account.
  * a.k.a inter-procedural analysis.

  ![Interprocedural Analysis](https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/19.png)
  
_Speaker Notes:_

In the context-insensitive analysis we analyze all functions once independently of calling context (we don’t care where the function was called from and what was the state at the point of invocation). Thus `f()` will be analyzed once, with no context, forcing the analysis to assume that input variable `x` can hold any possible value (a.k.a top `T`). Incrementing `T` by 1 results in `T`, which will be the return value of `f()`. This will be assigned to `y` in `L1`, resulting in the very imprecise abstract state on the left.

A context-sensitive analysis, will maintain the calling context, i.e., what values are possible for `x` at the callsite to `f()`. The analysis will carry the `{ y = 0 }` context to `f()` and plug that to the input argument `x`, resulting in a `{ x = 1 }` state for `f()` which in turn will be assigned to `y` at `L1`, resulting with the state on the right. This may remind you of inlining performed by compilers, as it indeed operates in a similar way.

## k-CFA

* k-CFA is a property of context sensitive analysis.
  * CFA stands for Control Flow Analysis.
* It basically describes the size of the context maintained throughout the analysis.
  * For interprocedural callsite-based analysis, a 1-CFA analysis maintains the context of the caller, when starting the analysis of a function (the callee).
    * The context is the state of the caller, at the point where the function was invoked.
  * 2-CFA includes the state of the caller, and the caller of that caller.
  * Etc.
 
Note: CFA is a misnomer (since control relates to branching in general and not necessarily function calls), but it stuck.

## Call Site Sensitive Analysis 

* A kind of a context-sensitive analysis that retains context by call sites.

<p align="center">
  <img src="https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/21.png" width='75%'/>
</p>

_Speaker Notes:_

This is one kind of a context sensitive analysis. 

The analysis remembers the (chain of) call-site(s) from which the currently analyzed function arrived from, and carries context, in the form of program state, from that call location to the entry point of the program.

The program state captioned as “1-CFA” is the result of a callsite sensitive analysis with a callsite chain of length 1. The analysis knows that it arrived at `L1` in `hey()` from `L2` in `foo()`, and therefore uses what is known about the state of `foo()` at `L2` for the input variable `i`. Since not much is known (`x` in an argument to `foo()` and  is passed directly to `hey()`), the state maintains that the value of result is the value of `x` from `L2`, plus 1.

The state captioned as “2-CFA” is the result of a 2-callsite-length chain analysis. This means that the analysis tracks the state of the calling function, as well as the state of the caller to the calling function. In the example, the state shown is for `hey()` at location `L1`, that was called from `L2` (in `foo()`), which in turn was called from `L3` (in `goo()`). 

As you may imagine, the longer the call chain for the k-CFA analysis, the (exponentially) more expensive it is. Once of the defining papers for creating an efficient interprocedural analysis is Precise interprocedural dataflow analysis via graph reachability.

## Object Sensitive Analysis

* A kind of a context-sensitive analysis that retains context by objects/allocation-sites.

<p align="center">
  <img src="https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/22.png" width='75%'/>
</p>

_Speaker Notes:_

A different approach for maintaining context in object oriented code is object sensitivity. Instead of maintaining a chain of states for callsites, we maintain a chain of calling objects, and their state. 

Objects are almost almost always identified by Allocation Sites (abbreviated as AS).

In the example we have two objects of type `A`, identified by their two allocation site `AS1` and `AS2`. The state shown on the left is the exit state for `foo()`. You can see that the state is composed of 2 disjunctions: one for when the object is `a1` allocated at `AS1`, and one for `a2` from `AS2`.

A 2-CFA analysis for object sensitive would maintain a chain of two calling objects (for instance if we had class `C` that allocated `B` objects and invoked `bar()`).

## Hybrid Callsite-Object Sensitive Analysis

* Joins both worlds, more precise, more expensive.

<p align="center">
  <img src="https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/23.png" width='85%'/>
</p>

_Speaker Notes:_

Note: Object sensitivity may be expressed via callsite sensitivity by treating the object as an argument for the invoked call.

## Field-Sensitive Analysis

* An analysis that distinguishes different fields of an object.

  ![Hybrid Callsite-Object Sensitive Analysis](https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/24.png)

_Speaker Notes:_

It may be hard to imagine, but some analysis are able to scale better (be faster) by treating all the fields in some object as the same field. This is called field-insensitivity.

The state on the left is a result of a flow-insensitive,  field-insensitive analysis. All the fields for the objects are indistinguishable and marked as `*`. The sub-state pertaining to `AS1` comes from the `a.x = 1` assignment, and the `AS2` substate comes from `a.y = 2`.

The state on the right is both flow and field sensitive.












