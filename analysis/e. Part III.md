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

For applications that relied on external modules for validation, like a proprietary DLL as is the case with Puzzleball 3D, there would exist an extra "hand off" phase in between the input and the logic block.

This flow can be visualized as follows:
```
User Input ➜ Bridge ➜ DLL ➜ Logic/Processing ➜ Output
```
Essentially, the application's main executable would act like a messenger, collecting the user input (through a prompt or button click) before passing it to the DLL in the "bridge" phase. The DLL then runs that input through some processing where a "Judge" function would ultimately decide whether it is valid. Finally, the DLL will construct an output based on the result of the processing.

### What is the Judge exactly?
The Judge is simply a routine that validates an input to determine if it is valid or invalid.

Since the logic for Puzzleball 3D's activation mechanism is likely located in ``ra.dll``, and the launcher is custom-built, there must exist a "bridge" phase or function that helps pass the collected input (which would be the unlock code in this case) to the DLL in order for the Judge function to run it through the "meat grinder" so to speak.

If we could locate this "Judge function" and modify it to return a desired result after the processing, we could bypass the activation mechanism.

Before that however, we should take a look at the custom-drawn UI a little closer.

### Custom Render Engines
It was not uncommon for applications in the early 2000s to implement their own "mini rendering engine" for constructing custom interfaces. This was done to make the application stand out and bypasses the standard Windows controls in order to provide an improved experience. A great example of this is the popular (at least back then) media player app called Winamp.

When we throw this custom render engine into the mix, the ``Story`` of an application's validation mechanism changes slightly.

Instead of the DLL simply constructing and reporting the output at the end, it will attempt to communicate with the function actually responsible for constructing the appropriate output based on an internal ID.

Let's say the result is "Unrecognized Code" instead of "Wrong Code" which carries a specific ID of 101. This ID will be passed from the DLL, through a bridge, to the function responsible for initiating and constructing the appropriate output message that might state something like "Unrecognized unlock code entered. Please check your input".

Since the text for the error message is found in the ``Arcade.dat`` file, there must exist a "Resource Manager" in either the main .EXE or ``ra.dll`` that will act as a "Librarian" which will grab the correct error string to display based on the internal ID mentioned previously.

### What is the Librarian exactly?
This is usually a function responsible for loading resources into memory and then constructing and maintaing a "map" of all the items. It then retrieves the required resource straight from memory through the use of pointers.

### Why load into memory instead of pulling straight from the disk?
This is largely due to the fact that in the 2000s era, hard drives were prolific. This storage medium was slow compared to system memory. And if an app had to call the Resource Manager in order to tell the Librarian to fetch a specific resource each time a user interaction (button click for example) was performed, the UI would incur stutters, leading to poor user experience.

### Tracking the Librarian
Knowing the location of the Librarian could come in handy when attempting to trace error messages through pointers (like the ID example) in memory.

Instead of setting a breakpoint on an error string, we could consider setting them on calls to Windows APIs that allow the application to communicate with resource files like ``Arcade.dat``.

Since the error strings like "Unrecognized Unlock Code" are located in an external file, ``Arcade.dat``, it would be more practical to set a breakpoint on function calls to APIs like ``CreateFile`` and ``ReadFile`` which are more than likely used to open the .dat file and pull the required string from it.

We can trace our way to the Librarian by:
1. Loading Puzzleball 3D through WinDbg.
2. Navigating to the input field in the launcher's sub menu.
3. Setting a breakpoint through WinDbg with the command ➜ ``bp kernel32!ReadFile``
4. Looking at the call stack when the breakpoint hits.
5. Finding the parent class ➜ This is the Librarian.

### How To Find the Judge
Let's go back to locating the Judge and the activation mechanism. As a quick reminder, the Judge will be function that:
- Receives user input.
- Performs math or processing.
- Determines if result is valid or invalid and sets a flag based on the result.
- Hands over execution to a Resource Manager and Librarian that will construct the appropriate error dialog.

There are several approaches we could take at this point in time to locate the Judge:
1. Tracing the error string, with help from the Librarian, to locate the routine that called for the mini rendering engine to draw the error dialog on screen.
2. Tracing the user input in memory to the first routine that "touches" it.

With option 1, there is a risk of getting tangled inside routines that are simply constructing individual elements on the launcher menu. It is easier to locate the Judge with option 2 as it cannot tell if a key is valid or invalid without reading the input first.

### Testing with WinDbg
First, we set a breakpoint on the ``ReadFile`` API with the command ``bp kernel32!ReadFile`` so that the application would "freeze" when we click the ``SUBMIT`` button as the error string is located in ``Arcade.dat``, which is an external file that needs to be read.

We then attempt to locate the entered user input by using the command ``s -a 0 1?80000000 "CAKE"``. This command basically searches the entire memory space for the ASCII string "CAKE" which is unmistakable.

<img width="615" height="219" alt="image" src="https://github.com/user-attachments/assets/95e5b57b-a642-4077-9fef-8ee1f5ddf068" />

We then set hardware breakpoints on all the memory addresses found where the user input resides. In this case, it was addresses, ``0248b081``, ``051e66d1``, and ``0520aac1``.

We then step forward in WinDbg to see which breakpoint gets hit and look at the call stack.

<img width="529" height="423" alt="image" src="https://github.com/user-attachments/assets/3cdae877-3e57-49ae-ba88-109d6ea6071f" />

The function, ``radll_GetUnlockCode`` seems highly likely to be the function that validates the user input, and hence, is the Judge.

Setting a breakpoint on this function should help us to confirm this. However, this breakpoint didn't end up triggering after relaunching Puzzleball 3D and going through the activation process by inputting a random word into the input field.

It turned out that the application was calling ``radll_GetUnlockCode`` through random pointers in memory, which made it difficult to locate the exact position to set a breakpoint.

I decided to try tracing to the Judge by locating the mini rendering engine, and then comparing the call stacks to narrow down on a common return address.

### Locating the Mini Rendering Engine

