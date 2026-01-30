# Part III
## A Shift
In the previous part, we saw how methodology alone could become insufficient when put up against a complex routine with little to no context. What worked well for uncovering the earlier validation mechanisms was quickly diminished in the face of a relatively opaque function that made even brute-forcing unfeasible.

The tools and techniques in Reverse Engineering are simply a means to an end. That end, is to be able to piece together the original developer's intent behind the design of a particular application.

Every piece of software can be thought to possess its own "Story"; an expected workflow from start to end, from input to output.

This is especially important to consider for local applications that don't strictly require server-side validation. This is because all the resources needed for the proper and complete functionality of the app is found within its binary and the system it is hosted or installed on.

It should only be a matter of time, effort, and most importantly understanding before every detail is unobfuscated, every "secret" unraveled.

With this new mindset, I set out to piece together the ``Story`` of Puzzleball 3D, starting with gathering historical context before eventually moving to renewed analysis, in order to better my attempts at cracking the activation mechanism.

## The Activation Mechanism Story
### Some history...
Back in the 2000s and prior, the ``Story`` of a particular application usually revolved around a specific and procedural path, especially with regard to its activation or validation system.

At it's core, the system would begin with gathering an input, applying some logic and/or processing to that input, and then constructing an appropriate output based on the result.

For applications that relied on external modules, like a proprietary DLL as is the case with Puzzleball 3D, there would exist an extra "hand off" phase in between the input and the logic block.

This flow can be visualized as follows:
```
User Input ➜ Bridge ➜ DLL ➜ Logic/Processing ➜ Output
```

