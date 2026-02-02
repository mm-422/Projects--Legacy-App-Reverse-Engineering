# Part III B
> GOAL: Locate the Judge.

As a quick reminder, the ``Judge`` will be a routine in the program that:
- Receives user input.
- Performs math or processing on the input.
- Determines if result is valid or invalid and sets a flag based on the result.
- Hands over execution to a ``Librarian`` that will construct the appropriate error dialog.

## Finding the Judge
Based on all the testing we've performed up to this point, there are two approaches we could take to locate the ``Judge``:
1. Trace the error string through the ``Librarian`` to locate the function that called for the "mini rendering engine" to draw the error dialog on screen.
2. Locate the user input in memory, set a breakpoint, and "catch" the first function that "touches" it.

With ``option 1``, there is a risk of getting tangled inside routines that are simply constructing individual elements on the launcher menu and have nothing to do with the activation mechanism.

With ``option 2``, we stand a higher chance of success as the ``Judge`` and "downstream" routines must be able to receive and read the user input first before it could determine if a key is valid or invalid. The function that attempts to access or read the user input stored in memory, soon after the ``SUBMIT`` button is clicked to initiate the activation process, is very likely to be closely related to the ``Judge``.

#### There is a problem
In order to locate the user input in memory, we need the application to be in frozen state. This means that we'll need to set a breakpoint somewhere in Puzzleball 3D's code which triggers as soon as the ``SUBMIT`` button is clicked and definitely before the ``Judge`` is able to get its hands on it.

This is where the unit testing function, ``unittest_ValidateUnlockCode`` comes in. While this routine was never utilized by Puzzleball 3D during normal operation, I wondered if there existed a similar looking function somewhere in the main .EXE or DLL file that performed the logic for activation.

We know that ``FUN_10018172`` and ``FUN_100180d1`` are the functions that likely perform some form of processing on an input in order to validate it.

I searched for XREFs to these functions and was able to find a function in ``ra.dll`` called ``FUN_1000B555``.

### ♦️ Testing with WinDbg
First, we set a breakpoint on the ``ReadFile`` API with the command ``bp kernel32!ReadFile``. This should cause the application to "freeze" when we click the ``SUBMIT`` button.

This is because the error strings are located in ``Arcade.dat``, which is an external file that needs to be read through ``ReadFile``.

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
After extensive tests and call stack comparisons, these are the routines involved:
```
➜ UI/Frame Construction Flow:
radll_DrawNextFrameIntoBuffer => 10007f07 => 10006fcb => 100c3444 => radll_IsASystemUpdateRequired => radll_IsTheMenuSessionComplete => radll_GetNumberOfRectsToUpdate => radll_GetUpdateRect => radll_HandleWindowsMessage =>
(Eventually hits, runs continuously) 10098bca

➜ KEY Validation Flow (sub-menu access by clicking link, NOT KEY SUBMIT):
10091408 => 1008fcaf => 100b62cc
=> UI/Frame Construction Flow (updates menu to draw sub-menu) =>
(Eventually hits, runs continuously) 10098bca

➜ KEY Validation Flow (On clicking activation menu to make active, NOT KEY SUBMIT):
10091408 =>1008fcaf => 100b62cc => 1000b4da => UI/Frame Construction Flow

➜ KEY Validation Flow (ON KEY SUBMIT):
10091408 => 1008fcaf => 100b62cc => 1000b4da => | 100b53e8 => 1000286c => 1000b555 => UI/Frame Construction Flow (updates menu to display error message box).
 
➜ On Clicking OK Button (to close error message box):
10091408 => 1008fcaf => 100b62cc => 1000b4da
=> UI/Frame Construction Flow (updates menu UI to display sub-menu again).
```

At this point, the Judge might very well be either ``FUN_1000286C`` or ``FUN_1000B555``.

### Confirming the Judge (``FUN_1000286C`` vs. ``FUN_1000B555``)
In a typical "game loop", the application has a flow that looks like this:
1. The Event Handler: Gathers the user input when a trigger (like a ``SUBMIT`` button) is clicked.
2. The Judge: Takes the string and moves to determine if it is valid.
3. The Logic: Receives string from the Judge and performs math.

Based on the Key Validation Flows above, ``FUN_1000286C`` is the most likely candidate for the Judge.

### How to prove 1000286C is the Judge?


