# Part III A
In the previous part, we saw how methodology alone could become insufficient when put up against a complex routine with little to no context. What worked well for uncovering the earlier validation mechanisms, became quickly diminished in the face of a relatively opaque function that made even brute-forcing unfeasible.

We should remember that **no tool or technique** in Reverse Engineering is a _one-size-fits-all_ type of solution. They are simply a means to an end, which is to piece together the original developer's intent behind the design of an application.

This "pieced-together intent" is what I like to call the ``Story``.<br>
Essentially, this is the expected flow of an application and/or its components from start to end. It is not just about **what** the application does, but **how** it does something.

Gaining a solid understanding of an application's ``Story`` will allow us to be more measured and accurate when analyzing binaries as well as reduce the likelihood of falling into "self-made traps" that might stem from bad assumptions. 

## The Activation Mechanism Story
For local applications like Puzzleball 3D that don't strictly require any server-side verification, the ``Story`` for its activation/validation mechanism might be as simple as gathering an input, applying some math or processing, and then producing an output based on the result.

Since all the resources needed for the proper and complete functionality of the application is contained within its binary and the system it is hosted or installed on, it should only be a matter of **time, effort, and most importantly understanding,** before every detail is unobfuscated and every "secret" unraveled.

## Gathering Historical Context
In order to flesh out the ``Story`` for Puzzleball 3D's activation mechanism, we first need to get into the mind of the original developer(s). This requires some research on period-relevant information, hence the "historical context".

### ♦️ Early Application Design
Back in the 2000s, the ``Story`` of a particular application usually revolved around a specific and procedural path, especially with regard to its activation/validation system.

At it's core, the system would begin with gathering an input, applying some logic and/or processing to that input, and then constructing an appropriate output based on the result.

For applications that relied on external modules for validation routines ― like a proprietary DLL in the case of Puzzleball 3D ― there would exist an extra "hand off" phase or "bridge" in between the input and the logic block.

This flow can be visualized as follows:
```
User Input ➜ Bridge ➜ DLL ➜ Logic/Processing ➜ Output
```
Essentially, the application's main executable would act like a **messenger**, collecting the user input ― like through a prompt, button click, and so on ― before passing it to the associated DLL with the help of a "bridge" function.

The DLL then runs that input through some processing where a "Judge" function would ultimately decide whether it is valid. Finally, the DLL will construct an output based on the result of the processing.

### ♦️ What is the Judge?
``The Judge`` is simply a routine that determines if an input is valid or matches an expected output.

Since the logic for Puzzleball 3D's activation mechanism is likely located in ``ra.dll``, and the launcher is custom-built, there must exist an intermediate function or "bridge", that helps pass the collected input ― which would be the unlock code in this case ― to the DLL in order for the ``Judge`` to run it through the "meat grinder" so to speak.

If we could locate the ``Judge`` and modify it to return a desired result **after** the processing, we could bypass the activation mechanism.

Before that however, we should take a look at the custom-drawn UI a little closer.

### ♢ Custom Render Engines
It was not uncommon for applications in the early 2000s to implement their own "mini rendering engine" for constructing custom interfaces. This was done to make the applications stand out and bypasses the standard Windows controls in order to provide a potentially improved experience. A great example of this is the popular media player app (at least back then) called Winamp.

When we throw this "custom render engine" into the mix, the ``Story`` of an application's validation mechanism changes slightly.

Instead of the DLL simply constructing and reporting the output at the end, it will attempt to communicate with the function actually responsible for constructing the appropriate output based on an internal ID.

Let's say the result after processing an unlock code is ``Unrecognized Code`` instead of ``Wrong Code`` ― both being different but essentially meaning "invalid code" ― which carries a specific internal ID of 101. This ID will then be passed from the DLL, through possibly another bridge, over to the function responsible for constructing the message tied to that ID ie. _Unrecognized unlock code entered. Please check your receipt or contact support._

Since the error text is found in the ``Arcade.dat`` file, there must exist a routine in either the main .EXE or ``ra.dll`` that will act as a "Librarian" to grab the correct error string to display based on the internal ID mentioned previously.

### ♢ What is the Librarian?
This is usually a function responsible for loading items/resources into memory and then constructing a "map" to keep track of them. It then retrieves the required item straight from a location in memory through the use of pointers.

Why load those items into memory instead of pulling them straight from the disk?
This is largely due to the fact that hard drives were prolific in the 2000s. This storage medium was much slower than system memory. And if an application had to call the ``Librarian`` to fetch a specific resource each time a user interaction ― like a button click ― was performed, the UI would incur stutters and lead to poor user experience.

This is a critical bit of understanding that helps make the flow of Puzzleball 3D's components seem clearer.

### ♢ Tracking the Librarian
Knowing the location of the ``Librarian`` could come in handy when attempting to trace error messages through pointers in memory.

Instead of setting a breakpoint on strings found in binaries, we could consider setting them on calls to Windows APIs that allow the application to communicate with external data/resource files like ``Arcade.dat``.

Since error strings like ``Unrecognized Unlock Code`` are found in ``Arcade.dat`` ― an external file ― it would be more practical to set a breakpoint on API calls like ``CreateFile`` and ``ReadFile``, which are more than likely used to open ``Arcade.dat`` and read from it before pulling the required string.

We can trace our way to the ``Librarian`` by:
1. Loading Puzzleball 3D through WinDbg.
2. Navigating to the input field in the launcher's sub menu.
3. Setting a breakpoint through WinDbg with the command ➜ ``bp kernel32!ReadFile``
4. Looking at the call stack when the breakpoint hits.
5. Finding the parent class ➜ This is the ``Librarian``.
