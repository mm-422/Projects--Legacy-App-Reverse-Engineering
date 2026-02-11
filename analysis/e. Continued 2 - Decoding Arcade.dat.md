# Part 2 Continued 2
With the integrity checks for the main executable and the DLL both bypassed, Puzzleball 3D's files should now be free to modify without worry of breaking app functionality. To confirm this, we can revisit the previous attempt to narrow down the source for the ``Fatal Not Found`` error dialog's text elements.

<img width="445" height="446" alt="list sucks" src="https://github.com/user-attachments/assets/bb3eb745-96b8-427f-bb23-20dbb9423acd" />

As can be seen above, the change made to the string in Part II is now reflected in the dialog box. This proves that the error message's text is indeed **pulled from the** ``ra.dll`` **binary.**

While this does not solve the actual problem causing the error in the first place, it does help give direction for the next step; to examine ``ra.dll`` for a way to fix the  ``Fatal Not Found`` error with the confidence that there are no integrity checks or "noise" now that might get in the way.

## Bypassing Arcade.dat Check
>GOAL: Disable or fix "Fatal Not Found" error.

As mentioned previously, the wording in the above error dialog indicates that the information was likely intended for the original developers of Puzzleball 3D, not the end user. There are also bizzare details like ``Arcade Main 1024`` which is not a file or path that exists on the system. There is no apparent "Fonts" list or "Resources" folder in the root directory either but we will get back to this in Part III.

At this point in the project, if there is a mechanism for validating the ``Arcade.dat`` file before it is loaded into memory perhaps and used for the launcher menu's construction, it could be similar in structure to the "integrity-check" implementations seen previously for the main .EXE and the DLL.

I began by searching both of the binaries for references to ``Arcade.dat`` and ``Arcade Main 1024`` among others. I then attempted to trace them "up" to a parent function in order to determine an appropriate breakpoint location. Below is a snippet of some of the tests:

```
String 1 ➜ "The game is looking for some data or a file..."
    • Searching for this in the DLL led us to LAB_004168B3.
    • There is an exact duplicate of this string in the main executable.
    • Patching JNZ LAB_10082432 => JMP LAB_10082432 under function FUN_10082675 causes the app to crash when "Already Paid" button is pressed to access the launcher's sub-menu.
    • There is another address in the DLL, 1008267B, where a similar function block can be found.
    • Modifying this DID NOT seem to produce an effect.

String 2 ➜ "Arcade.dat"
    • There is one reference in the DLL at 10015C50.
    • Not much is revealed other than that the parent function, FUN_10015C0D, returns a 1-byte bool in AL.
    • There are several similar-looking function blocks for ra.dll, Application.dat, Channel.dat and so on.
    • Modding the FUN_10015C0D block to mimic expected flow (as with original Arcade.dat) DID NOT cause an effect.

String 3 ➜ "Arcade Main 1024"
    • Searching for this led to LAB_10006A5B which is a block under FUN_1000697E.
    • FUN_1000697E has one parent XREF => radll_EnterMenuSession.
    • From testing, we know that setting a breakpoint at radll_EnterMenuSession actually triggers when the button to access the launcher sub-menu is clicked.
    • This is true for both genuine and modified Arcade.dat files.
    • This is an indicator that the mechanism that does validation for Arcade.dat may be located somewhere in the radll_EnterMenuSession sub-routines.
```
Suffice to say, most of the leads discovered for testing at this phase led to dead ends. Some instruction modifications had no observable effect while others caused Puzzleball 3D to crash on pressing the ``Already Paid`` button.

After much trial and error, I discovered a sub-routine called ``FUN_1007F63F`` in the DLL, which through preliminary observation with WinDbg seemed to perform a "loader" type of functionality for components of Puzzleball 3D like the ``Application.dat``, ``Channel.dat``, and of course, ``Arcade.dat`` resource files that are all found in the root directory.

#### FUNCTION 1007F63F
<img width="524" height="239" alt="arcadeloader" src="https://github.com/user-attachments/assets/b56e59ff-bb40-4994-8c7d-a59d84e9f62e" />

The above is only a snippet of the function's beginning section. ``FUN_1007F63F`` is massive with numerous branches of sub-routines that loop over each other, where even the decompiled view would likely take up multiple pages worth of space on this repo. 

Attempting to decode this function with the previous methodology of making comparisons between the expected and actual values in memory, registers, etc. and correlating that information with context from stepping through Puzzleball 3D's code with WinDbg, was not efficient.

I eventually understood that this function was performing an extremely long routine (hundreds of loops) of enumerating system details alongside loading almost every "piece" of each component (``Application.dat``, ``Channel.dat``, ``Arcade.dat``, etc.) that was used to draw the launcher menu's elements; everything from text box dimensions to button colors.

It was simply not feasible to trace each loop in hopes of discovering the chain that would lead to the typo. I deemed this approach another "failure" as performing such "grunt work" with no real certainty of the outcome would likely lead to endless frustration.
