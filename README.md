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

‘x = nothing’ means that x is not initialized so we don’t have any valid possible value for it. Another notation for this is x = ⟂ (also called ‘bottom’).

Note: some analysis treat uninitialized variables as having *all possible values* (since this may be the case for some languages like C), which is usually denoted as x = ⟙ (also called “top”)

  
  
