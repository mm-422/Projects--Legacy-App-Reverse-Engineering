# Part II A
One crucial step I missed in Part I was to verify if the function, ``unittest_ValidateUnlockCode``, was even a relevant part of Puzzleball 3D's binary used during any actual code validation process ie. when inputting an unlock code and clicking submit.

If we looked closer to examine the name of the function alone, the word "unittest" here is a hint. 
In software programming, "unit testing" is a method for verifying if individual components of an application are working as expected. In this case, the ``unittest_ValidateUnlockCode`` function block is likely tied to ensuring that user input validation (like an unlock key) executes as expected.

This does not necessarily mean that the same function is also used for validating "real" user input from the main application. Often times, it is good practice to design isolated code blocks that can be automated for unit testing. But as we will see later, this isn't always the case with Puzzleball 3D.<br>

## A New Approach
> GOAL: Verify validity of unittest_ValidateUnlockCode.

Determining the relevancy of ``unittest_ValidateUnlockCode`` is not something that could be done through static analysis alone. To verify an application's behavior would typically require "stepping through code" with a debugging tool.

To do this, I used x64dbg to set a memory breakpoint on ``unittest_ValidateUnlockCode``, and then proceeded to go through the activation process within Puzzleball 3D's launcher menu.

### ✦ Hypothesis
If the memory breakpoint in x64dbg is triggered, then we could revisit the function with a different approach. Otherwise, we would need to trace the user input to the function that **actually performs the activation** or validation step.

## x64dbg Testing
Setting a memory breakpoint in x64dbg can be done through the "Symbols" tab. This is where all of the modules (other libraries needed for app's functionality) are listed along with their imported and exported "symbols" which include variables, data structures, and functions.

<img width="1280" height="720" alt="x64dbg symbols" src="https://github.com/user-attachments/assets/d8a44470-7d96-4bf5-b13d-35d4b1f95dae" />

As we've established earlier, ``ra.dll`` is the binary where the protection mechanism is likely located and is where the ``unittest_ValidateUnlockCode`` function is found under the Symbols tab as expected. 
To set a breakpoint here, we simply right click on the target function and select "Toggle Breakpoint" or press the F2 shortcut key.

This breakpoint will then be listed under the "Breakpoints" tab in x64dbg with a "hit count" next to it to indicate the number of times the breakpoint was triggered during app runtime.

Now, all that's left to do is to carry out a test to see if the hypothesis is true or false. I navigated to the sub-menu shown previously past the ``Already Paid`` button, then clicked on the hyperlink to bring me back to the window where the product key was displayed along with an input field for an unlock code.

<img width="1280" height="720" alt="not connected" src="https://github.com/user-attachments/assets/3679889d-e5f6-488c-bb22-a96da99c0c9a" />

I entered a random string of characters (234A) and then clicked the ``SUBMIT`` button. If the ``unittest_ValidateUnlockCode`` function was indeed tied to the activation mechanism here, then the  breakpoint set in x64dbg should have been hit and the hit counter increased by 1.

It is also typical for the application being debugged to appear "frozen" or "halted" when this happens because of its execution being "held in place" at the breakpoint, so to speak, by the debugger (x64dbg in this case).

Unfortunately, neither of these events occurred and all I got was a pop-up window telling me that I've entered an "unrecognized unlock code".

<img width="1280" height="720" alt="Unrecognized Code" src="https://github.com/user-attachments/assets/600f91fa-bf4d-460c-a0c8-37288203732d" />

With this, we can confirm that ``unittest_ValidateUnlockCode`` **is not involved** in the activation process. It could even be vestigial code from early development or debugging that isn't necessarily reflective of the actual math or logic behind the real activation mechanism of Puzzleball 3D.

Now we need to find the actual activation mechanism or function responsible for validating unlock codes.

## Recalibrating
> GOAL: Locate the routine responsible for activation.

At first, I tried a "brute-force" approach by setting breakpoints on all the "named" functions listed under ``ra.dll`` as exports in the Symbols tab.

I then proceeded to repeat the activation process through the launcher menu.

While some breakpoints for certain functions were hit at the startup of Puzzleball 3D, like even ``unittest_GetBrandedApplicationID``, the result upon clicking the submit button was the same. An "unrecognized unlock code" message box.

So then, if the function responsible for validating unlock codes was not one of the exported ones from the DLL file, could it be located in the main application executable? This was an assumption that I had that led me on a long and frustrating goose chase that ultimately ended with nothing substantial. After some research, I decided to try tracing the user input again but with a different method involving **Windows Messages.**

### ✦ An Intro to Messages
**Windows Messages** are units of data used to communicate events - _like mouse clicks and keyboard presses_ - between the OS and applications.

Each application window in the **Windows** operating system is placed under an "Application Thread" that contains an internal queue of these events. Each event can be thought to possess a label like ``WM_LBUTTONDOWN`` for a mouse click.

If this mouse click happened after a keyboard press (``WM_KEYDOWN``) for example, then you can visualize the event queue as being a chronological order like this:<br>
> ``WM_KEYDOWN``⤵︎<br>
> ``WM_LBUTTONDOWN``

### ✦ The Message Flow
Typically, most if not all elements in a modern app (like buttons and text boxes) are their own "windows" in that they have a unique ID or handle called an ``HWND`` (Window Handle). An example of this might be something like, ``00451233``.

A function called ``GetMessage`` within these apps runs a "Message Loop" that keeps track of all these events and handles. The typical flow from user input to app output is as follows:

1. You click a button, the kernel identifies the ``HWND`` of the window right beneath the mouse cursor.
2. The OS then sends a message (``WM_LBUTTONDOWN``) along with the corresponding HWND to the app's event queue.
4. ``GetMessage`` will see this event and pass it to another function called ``DispatchMessage``, which performs a check - _tracing the ``HWND`` to the specific window like a button_ - before handing over to the corresponding window's ``WNDPROC`` or Window Procedure.
5. ``WNDPROC`` is a "central" callback function that contains the logic for the window. This is where the execution steps following a specific event is initiated. For example, a mouse click causing a part of the app to change color or launch a new instance.

Developers typically override or customize the ``WNDPROC`` function and its name to suit the intended functionality.

### ✦ Hypothesis
If we could find the exact name of the ``WNDPROC`` implementation in Puzzleball 3D and trace the ``HWND`` of the user input field or "SUBMIT" button through the application's code, we might be able to locate the core activation mechanism.

## Spy++ Testing
We can utilize a tool called Microsoft Spy++ to easily uncover the properties of a target window such as the HWND and parent class of a button.

To do this, we simply locate the process with the application's name (Puzzleball 3D in this case) within Spy++ and open the Messages window. Here we can see a long list of events:

<img width="1280" height="720" alt="spy++ test" src="https://github.com/user-attachments/assets/6d628381-0d40-4285-8b08-e319187d8f7d" />

We then perform an interaction with the app - _like clicking a button_ - and then locate the corresponding event from the list. Puzzleball 3D's launcher also seems to run a continuous "hit test" as can be seen with the numerous ``WM_NCHITTEST`` messages. I didn't think too much of this at the time but the reason will become apparent in a later part.

Anyway, the ``HWND`` has been identified as ``00080428``. However, pointing or clicking on any space within the launcher window (apart from the top menu bar) with the "Finder Tool" in Spy++ gives us the same ID as well.

I restarted Puzzleball 3D and performed the same test. The handle changes with each run but stays constant for all elements displayed on the launcher window. This was puzzling. 

<img width="1280" height="720" alt="spy++ inspect" src="https://github.com/user-attachments/assets/9fd7f959-e003-465a-b604-ba1f114141fb" />

## Another Curveball



