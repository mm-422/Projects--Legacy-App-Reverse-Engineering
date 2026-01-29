# Part II C
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
String 1 ➜
    • Looking for the string "The game is looking for some data or a file..." leads us to LAB_004168b3.
    • There is an exact duplicate of this string and code in the main app executable as in the DLL.
    • Modifying the main app seems to not produce any effect, as before with the DLL verification step.
    • Performing similar mods in the DLL (JNZ LAB_10082432 => JMP in FUN_10082675) causes the app to crash when button is pressed to access activation window.
    • There is another address in the DLL [1008267b] where a similar function block can be found. Modifying this did not seem to produce an effect.

String 2
- Looking for the string "Arcade.dat" did not reveal anything in the main app. However there is one reference in the DLL at [10015c50].
    • Not much is revealed other than the parent function, FUN_10015c0d, returns a 1-byte bool in AL, and that it has several similar blocks for ReflexiveArcade.dll, Application.dat, Channel.dat and so on.
    • Modding the entire FUN_10015c0d block to follow expected flow did not seem to cause an effect.

String 3
    • Searching "Arcade Main 1024" leads to LAB_10006a5b which is a block under FUN_1000697e.
    • FUN_1000697e has one parent XREF => radll_EnterMenuSession.
    • From testing, we know that setting a breakpoint at radll_EnterMenuSession actually triggers when the button to access activation window is clicked.
    • This is true for both genuine and modified Arcade.dat files.
    • This is an indicator that the mechanism that does validation for Arcade.dat may be located somewhere in the radll_EnterMenuSession sub-routines.
    • This is our next point of focus.
```

Suffice to say, almost all of the testing done at this phase led to dead ends. instruction modifications that either caused the app to crash or not reflect any change whatsoever.

Eventually, through observation and tracing with Ghidra and WinDbg, I landed on a sub-routine called ``FUN_1007F63F`` in ``ra.dll`` which seemed to perform a "loader" type of functionality. I also decided to start labelling some of these functions to make keeping track of them easier.

#### FUNCTION 1007F63F
<img width="524" height="239" alt="arcadeloader" src="https://github.com/user-attachments/assets/b56e59ff-bb40-4994-8c7d-a59d84e9f62e" />

The above is only a snippet of the routine's beginning section. ``FUN_1007F63F`` is simply massive with numerous branches of sub-routines that looped over each other, where even the decompiled view would likely take up multiple pages worth of space on this repo. 

Attempting to decode this with the previous methodology of following the application's flow through WinDbg and making comparisons between expected and actual values in memory, registers, etc. was far too arduos. 

I eventually understood that this function was performing an extremely long routine of enumerating system details alongside each and almost every element that was drawn on the launcher menu. This was meant dozens if not hundreds of similar looking loops with little context.

I had to deem this approach a "failure" too because it simply wasn't practical or even feasible to painstakingly track an application's flow hundreds of times with no real confirmation of what it is trying to do.
