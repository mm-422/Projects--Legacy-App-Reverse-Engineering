# Part III B
> GOAL: Locate the Judge.

As a quick reminder, the ``Judge`` will be a routine in the program that:
- Receives user input.
- Performs math or processing on that input.
- Determines if the input is valid or invalid and sets a flag based on the result.
- Hands over execution to a ``Librarian`` that will construct the appropriate error dialog.

## Finding the Judge
There are 2 ways to locate the ``Judge`` based on all the testing performed up to this point:

### ♦️ Option 1
```
• Trace the error string in the DLL back up to the Librarian.
• Locate the internal ID for the error, if there is one.
• Follow this ID back to the function that calls for the "mini rendering engine".
• This function likely receives a command from the Judge to draw the error dialog.
• The Judge should be close.
```

### ♦️ Option 2
```
• Load the program into WinDbg.
• Enter a fake unlock code into the launcher menu.
• Locate the code in memory and set a hardware breakpoint on read.
• Step forward in WinDbg.
• "Catch" the first function that accesses it.
• This function likely hands over the user input (code) for processing.
• The Judge should be close.
```

With ``option 1``, there is a risk of getting tangled inside routines that are simply constructing individual elements on the launcher menu and have nothing to do with the activation mechanism.

With ``option 2``, we stand a higher chance of success as the ``Judge`` and "downstream" routines must be able to receive and read the user input first before it could determine if a key is valid or invalid.

The function that attempts to access or read the unlock code stored in memory, soon after the ``SUBMIT`` button is clicked which initiates the activation process, is very likely to be closely related to the ``Judge``.

### ♦️ Extra Consideration for Option 2
The second method actually has a prerequisite in that we need the application to be in a frozen state before we could sift through the memory for the user input. This means that we'll need to set a breakpoint somewhere in Puzzleball 3D's code which triggers as soon as the ``SUBMIT`` button is clicked and definitely before the ``Judge`` is able to get its hands on the unlock code.

This is where the earlier unit testing function, ``unittest_ValidateUnlockCode`` comes in.
In Part I, we confirmed that this function is never ran through or utilized by Puzzleball 3D during normal operation. But function blocks for unit testing often take a similar form to the "real" routine that is actively used.

<img width="1280" height="722" alt="the 2 functions" src="https://github.com/user-attachments/assets/c9265719-2080-4e0d-bb19-d4a47d498de3" />

We know that the functions, ``FUN_10018172`` and ``FUN_100180d1``, in ``unittest_ValidateUnlockCode`` likely perform some form of processing on an input in order to validate it. These are functions that serve a specific purpose and are likely only found in one other routine if any.

I searched for references to these functions and was able to find a parent routine in ``ra.dll`` called ``FUN_1000B555`` that contained calls to both of these functions.

<img width="1218" height="627" alt="1000b555 assembly" src="https://github.com/user-attachments/assets/74b1192a-1c82-46cf-bbb2-13a7f4ad8022" />

After setting a breakpoint at the beginning of ``FUN_1000B555``, I launched Puzzleball 3D again through WinDbg and attempted to go through the activation process. Fortunately, clicking on the ``SUBMIT`` button now causes the application to freeze, indicating that the breakpoint was indeed hit in WinDbg.

<img width="1259" height="720" alt="windbg bp" src="https://github.com/user-attachments/assets/5cd09570-293a-4272-a31d-16c413079e8b" />

### ♦️ Hypothesis
Now that we have a reliable breakpoint to use for "freezing" the application right at the start of what is likely the activation/validation routine, we can most likely work our way to the ``Judge`` with some help from WinDbg.

If we carry out the testing outlined in ``Option 2`` and find any overlap in function calls/references between this and ``FUN_1000B555``, then we have found the ``Judge`` or a function very close to it.

## Option 2 Test with WinDbg
First, we set a breakpoint on ``FUN_1000B555``. This should cause the application to "freeze" when we click the ``SUBMIT`` button after entering an unlock code as demonstrated previously.

<img width="716" height="137" alt="CAKE input" src="https://github.com/user-attachments/assets/fb3889cc-4dfd-4049-aacc-4f0cb7015f3b" />

We then attempt to locate the entered user input by using the command ``s -a 0 1?80000000 "CAKE"``. This command basically searches the entire memory space for the ASCII string "CAKE", which is a word I decided to use to make the result obvious and unmistakable, so that we don't accidentally look for similar but unrelated strings.

<img width="615" height="219" alt="cake windbg" src="https://github.com/user-attachments/assets/24e4b9e0-db08-4347-891b-cce54937d66f" />

We then set hardware breakpoints on all the memory addresses found where the user input resides and/or has been copied to. In this case, it was addresses, ``0248b081``, ``051e66d1``, and ``0520aac1``.

We then step forward in WinDbg to see which breakpoint gets hit and look at the call stack; paying special attention to the return address column.

<img width="528" height="423" alt="call stack overlap" src="https://github.com/user-attachments/assets/a6f82028-f87f-4d92-8fcc-2cfb79a48288" />

The return address for the item on top of the stack is ``1000B565`` which is actually located in ``FUN_1000B555``. If we look at the second item on the call stack, we see the return address, ``10002915`` which is located in ``FUN_1000286C``, the parent function of ``FUN_1000B555``.

<img width="933" height="137" alt="290c" src="https://github.com/user-attachments/assets/2c8ed547-cb98-4139-9553-e55a16b30bbe" />

The address ``10002915`` is right where the TEST instruction is. This indicates that the value in the AL register acts as a "flag"


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


