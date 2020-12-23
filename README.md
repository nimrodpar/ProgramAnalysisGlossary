# Program Analysis Glosarry

[I](https://nimrodpar.github.io/) created this glossary due to a recent program analysis exploration task. I found myself struggling to keep up with the jargon and all the (many) different kinds of analyses.

<h1 align="center">
  <a href='https://docs.google.com/presentation/d/1DkInjfdAT6BGIMI2xHavxGgLqrV-seBDvNx0gUgU-6c/edit?usp=sharing'> -->Slides<---</a>
</h1>

The slides were created first, but I'm in the process of transcribing them directly to the readme here.

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
  
  ![A Program Analysis](https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/4.png)
  
_Speaker Notes:_

For the sake of simplicity, we define a very basic notion of a program analysis. The example we show here and in the next 3 slides is an interval analysis, which simply tried to track the range of values for variables.

`x = nothing` means that x is not initialized so we don’t have any valid possible value for it. Another notation for this is `x = ⟂` (also called ‘bottom’).

Note: some analysis treat uninitialized variables as having *all possible values* (since this may be the case for some languages like C), which is usually denoted as `x = ⟙` (also called “top”)

  ![A Program Analysis](https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/5.png)
  
_Speaker Notes:_

The abstract states here does not seem abstract at all right? The values are very concrete and well defined. 

This is because of:
  1. the very simple program we are analysing 
  1. the “abstract domain” we are using. 

An abstract domain simply keeps track of certain properties for the (variables in) the program. In this example the properties are integer values, but can also be stuff like “is the variable null or not?” or “is the value of the variable odd or even”?. 

Abstract domains in program analysis have many rules that are required for correctness and are a very big and deep field of study. We won’t elaborate any further, just remember that their purpose is to keep track of (properties of) variable values. 

## Definition: The Join Operation ⨆

* An operation for...joining abstract states flowing from multiple paths.
* Also called a "May Analysis"

  ![The Join Operation ⨆](https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/6.png)
  
_Speaker Notes:_

So, what do we do in the case of the pos() function? What is the abstract state at the `L3` location (i.e., what are all possible values for the variable `x`)?

Since we want to cover all possible values of variable `x` (this is also called “over-approximating”), we need to account for the values flowing into `L3`. The values are determined by the two branches of the if statement. So, we somehow need to account for the states that we have for `L1` and `L2`.

  ![The Join Operation ⨆](https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/7.png)
  
_Speaker Notes:_

The operation for accounting for all states flowing into a program location is called a Join operation (denoted ⨆)

Basically the operation means “the value for `x` in `L3` can be either the value of `x` in `L1` or the value of `x` in `L2`”. 

Here, we join by simply ORing all the possible states flowing in. But this can be done differently. For instance, instead of 2 values for `x` in two sub-states, we can have `{ x = [0,42] }` i.e., `x` can be any value between 0 and 42. Since we are over-approximating, this result is okay but also very inaccurate. 

This sort of compromise (also called abstraction) is needed for efficiency. One of the major goals of program analysis is to create efficient and scalable analyses that are still accurate enough to be useful.

## Definition: The Meet Operation ⊓

* Merging abstract states flowing from multiple paths while keeping facts that are *true on all paths*.
* Also called a "Must Analysis"

  ![The Meet Operation ⊓](https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/8.png)

_Speaker Notes:_

For some analyses, we do not want to account for all possible states on all paths. 

That’s the case for the the available expressions analysis shown in the slide. This is an analysis used by compilers to determine what temporary computations can be re-used and need not be calculated.

In the example, the expressions `2*y` and `3*x` are used multiple times so it would be wise to re-use the calculation if possible. The analysis tracks the computed expressions by keeping tabs on the inputs to each expression to check if it had changed. 

For instance, `2*y` is computed in L0 and is therefore added to the state at the location (the state is the list of available expressions). At L1 location, `2*y` is no longer available since `y` changes and is removed from the state (but kept in `L2`).  `3*x` is added to the state at `L1` and `L2` since it was computed in the if condition and `x` does not change in any of the branches. 

Arriving at `L3`, to correctly know which expressions are available and need not be recomputed, we must only consider the expressions that are available in all paths leading to `L3`. This means we must meet the states from `L1` and `L2`, and keep only the expression shared by both-- `3*x`.

# Analyses Types

## May Analysis

* (as seen before) an over-approximating analysis that uses the join ⨆ operation to merge states flowing from multiple paths.

  ![May Analysis](https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/7.png)

## Must Analysis

* (as seen before) an under-approximating analysis that uses the meet ⊓ operation to merge states flowing from multiple paths

  ![Must Analysis](https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/8.png)
  
## Forward-Analysis

* An analysis where the abstract state flows forward
  * The state at each program point is derived from the states of the **preceding** program points

  ![Forward-Analysis](https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/12.png)
  
_Speaker Notes:_

You may not have noticed, but the way states at each location were computed is by using the states from previous lines and accounting for what happens at the current line (also called applying a “transformer”).

This sort of analysis, where you use the states for (direct) previous lines, is called a forward analysis. Most analysis are forward ones, including what we saw so far.

  ![Forward-Analysis](https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/13.png)
  
_Speaker Notes:_

So that state from `L1` is taken as the initial state for `L2` (i.e., `in(L2) = L1`), and the operation `x=1` at `L2` is accounted for (the semantics of `L2` i.e., [| `x = 1` |]) and we get the resulting state for `L2` (i.e., `out(L2) = {x =1, y = nothing}`.

## Backward-Analysis
* An analysis where the abstract state flows backwards
  * The state at each program point is derived from the states of the **succeeding** program points
* Example: Live Variables analysis
  * An analysis used by compilers to determine which variables are no longer needed and can be freed

  ![Backward-Analysis](https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/14.png)
  
_Speaker Notes:_

This analysis starts the end of the program. It identifies that `L4` uses `y`, so `y` is live at `L4`.

  ![Backward-Analysis](https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/15.png)
  
_Speaker Notes:_

To determine which variables are live (i.e., required and can’t be freed) at `L3`, the state from `L4` is taken, and the statement in `L3` is examined. The analysis identifies that `x` is needed at `L3`, so it is added to the state.

Note that we already know that `y` can be freed after `L3`.

  ![Backward-Analysis](https://github.com/nimrodpar/ProgramAnalysisGlossary/blob/main/Slides/16.png)
  
_Speaker Notes:_

The final result of the analysis.










