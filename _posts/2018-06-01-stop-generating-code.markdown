---
layout: post
title:  "You (Probably) Shouldn't Be Generating Code"
date:   2017-12-23 7:42:15 -0500
categories: [random]
---

I've seen my fair share of strange phenomena in the realm of software engineering, but hardly
anything is more costly or insidious as the obsession with attempting to integrate **code
generation** as a "solution" to a given problem, despite how exciting or efficient it seems at
first.

# First Encounters

My first experience with a homebrewed code generation tool was immediately after graduating
college. I worked at a relatively small software shop that developed embedded GPS devices. Their
usage of code generation was thankfully restricted to their build system. The well-intentioned
tool in question was a couple hundred lines of Ruby that managed build/release configurations for
various customers and the output of said program was a massive `CMakeLists.txt` file that would be
processed by `CMake`, which, of course, generates `makefiles`.

Countless hours were wasted by various employees who tried to modify either the `makefiles` or the
`CMakeLists.txt` directly, only to find out that their changes are overridden during the
circuitous build process, which was kicked off by running `make`, ha!

Little documentation existed for this tool and the inner workings were intermingled with
the user configuration, so anyone who touched the tool would sometimes spend a full day (or more,
if the change was tricky) trying to understand it. This often led to the developer asking, "why
the **fuck** does this thing exist anyway?"

As an eager, budding software engineer facing my first real-world codebase, I was doe-eyed and
fascinated by the concept of generating the inputs to the build system. I sunk my teeth into it
and eventually had a good handle on why it existed and how to correctly modify it.

The original author was trying to prevent *Customer A* from seeing artifacts of *Customer B*'s
solution during the source distribution process, including artifacts that were only present in the
build system configuration. (They were also trying to manage which features should be turned on/off
during various development stages, but that can just as easily be done through CMake flags without
an unnecessary layer of indirection.)

However, since the custom code for a given customer (typically the hardware interface layer) was
segregated by directory, it seems that using the recommended CMake structure (rather than
generating a monolithic CMakeLists.txt) along with a separate release script (to choose the
correct build flags and directories for distribution) could have been an alternative solution.

I took three things away from this experience.

1. The more custom tools you write, the more documentation you're required to maintain. If the
original architect had managed to solve his/her problem with a standard configuration management
tool, the documentation would've been a freebie.
2. Code generation is extremely easy to introduce and really hard to eliminate. The path to
removing a code generation tool is almost never "check in the output and delete the tool" because
the output from the tool is probably "justifiably" insane due to the fact that it was generated in
the first place. This is why I consider code generation to be the *lazy* solution to a problem.
3. Layers of indirection are expensive. It takes the original developer time to develop a layer of
indirection, and it takes all other developers a period of time to understand it. It damn well
better buy you something *awesome*.

# True Source Code Generation

My next job was working for a government contractor with a true passion for **generating code**.
It is flat-out **assumed** by folks who don't write code (the people who are often calling the
shots at government contractors) that generating code is a cheap and smart solution. What they're
not considering is that the alternative to code generation isn't to write the code that *would be*
the output of a code generation tool (often repetitive, ugly, and unmaintainable), but rather **to
write organized and modular code that neatly segregates the pieces that ostensibly lend themselves
to generation** (the bits that change from one configuration to another).

I saw codebases from various projects at this company and their usage of code generation was
always something like this: They had a massive codebase that did not compile at checkout with
standard tooling. You had to *somehow* know how/when/where to run some tool that generates all of
the missing files required for compilation. These tools ingest some standard data format (if you
were lucky) and then spits out a .cpp or .h file with elements hardcoded in the tool itself and
other elements expressed as inputs.

Not only was it was a fucking **nightmare** to trace the origins of a given change, but the whole
concept could've been replaced by preprocessors (as much as I caution the use of macros, I would
highly recommend considering it as an alternative to code generation when it beckons you). I found
myself fixing bugs in lines of code that originated as strings of text in another piece of
software. That means that all of our great, modern tooling (syntax highlighting, linting, etc.)
were ineffective and that the test cycle for a given change now depended on the length of time it
took for this shitty tool to run.

This is all assuming that I've already found which of the tools is generating the code that
introduced the bug in the first place. Good thing they used **code generation** to prevent anyone
from having to write, debug, or maintain anything... right?

# Sometimes It Makes Sense

I'm not shitting on all forms of code generation. Sometimes, it really is great. For example,
Android's generated `R.java` file has probably saved hundreds of thousands of hours in developer
time by automatically generating resource IDs for all of your project's resources. (It probably
also caused you a few uncached headaches if you did any early Android development though.)

Also, any compiler/transpiler could be considered a "code generation tool" as well. I don't have
to convince you that these tools have been worth the trouble.

These forms of code generation, however, are beneficial because their developers are committed to
maintaining and documenting them. They also solve problems that **don't have simpler solutions**.
Their benefits far outweigh their costs. They are the exception to the rule that code generation
is hardly ever the right solution.

I hope this article was interesting. You can follow the discussion on reddit
[here](https://redd.it/8nwh1f).
