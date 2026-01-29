# Part II C
With the integrity checks for the main executable and the DLL both bypassed, Puzzleball 3D's files should now be free for modification without worry of breaking app functionality. To confirm this, we can revisit the previous attempt to narrow down the source for "Fatal Not Found" error dialog's text elements.



<img width="445" height="446" alt="list sucks" src="https://github.com/user-attachments/assets/bb3eb745-96b8-427f-bb23-20dbb9423acd" />

As can be seen above, the change to the string is now reflected obviously in the error dialog, confirming that the error message's text is drawn from ``ra.dll``. This of course does not solve the actual problem causing the error in the first place, but pursuing the original goal in a procedural manner did lead us to neutering two different integrity checks that would have gotten in the way sooner or later.

Now we can focus on fixing this "Fatal Not Found" error with the confidence that there are no integrity checks or "noise" getting in the way.

## Bypassing Arcade.dat Check
>GOAL: Disable or fix "Fatal Not Found" error.

As mentioned previously, the wording in the above error dialog indicates that it was likely intended for the original developers, not the end user. There are also bizzare details like ``Arcade Main 1024`` which is not a file, resource, or path that exists on the system. There is no apparent "Fonts" list or "Resources" folder in the root directory either (we will get back to this at a later part).

We need more clues at this point. If there is a mechanism for validating ``Arcade.dat``, it could be similar to the structure of the previous integrity checking mechanisms for the main .EXE and the DLL.

I began with searching both the main executable and ``ra.dll`` for references like ``Arcade.dat`` and ``Arcade Main 1024`` among others. Almost all of them led to dead ends; instruction modifications that either caused the app to crash or not reflect any change whatsoever.

Eventually, through observation and tracing with Ghidra and WinDbg, I landed on a sub-routine called ``FUN_1007F63F`` in ``ra.dll`` which seemed to perform a "loader" type of functionality. I also decided to start labelling some of these functions to make keeping track of them easier.

#### FUNCTION 1007F63F
<img width="524" height="239" alt="arcadeloader" src="https://github.com/user-attachments/assets/b56e59ff-bb40-4994-8c7d-a59d84e9f62e" />

The above is only a snippet of the routine's beginning section. ``FUN_1007F63F`` is simply massive with numerous branches of sub-routines that looped over each other, where even the decompiled view would likely take up multiple pages worth of space on this repo. 

Attempting to decode this with the previous methodology of following the application's flow through WinDbg and making comparisons between expected and actual values in memory, registers, etc. was far too arduos. 

I eventually understood that this function was performing an extremely long routine of enumerating system details alongside each and almost every element that was drawn on the launcher menu. This was meant dozens if not hundreds of similar looking loops with little context.

I had to deem this approach a "failure" too because it simply wasn't practical or even feasible to painstakingly track an application's flow hundreds of times with no real confirmation of what it is trying to do.
