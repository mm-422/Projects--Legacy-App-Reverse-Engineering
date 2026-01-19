# Part II
One crucial step I missed in Part I was to verify if the function, ``unittest_ValidateUnlockCode``, was even a relevant part of Puzzleball 3D's binary used during any actual code validation process ie. when inputting an unlock code and clicking submit.

If we looked closer to examine the name of the function alone, the word "unittest" here is a hint. 
In software programming, "unit testing" is a method for verifying if individual components of an application are working as expected. In this case, the ``unittest_ValidateUnlockCode`` function block is likely tied to ensuring that user input validation (like an unlock key) executes as expected.

This does not necessarily mean that the same function is also used for validating "real" user input from the main application. Often times, it is good practice to design isolated code blocks that can be automated for unit testing. But as we will see later, this isn't always the case with Puzzleball 3D.<br>

## A New Approach
> GOAL: Verify validity of unittest_ValidateUnlockCode.

Determining the relevancy of ``unittest_ValidateUnlockCode`` is not something that could be done through static analysis alone. Verifying an application's behavior typically requires "stepping through" the code with a debugging tool.

To do this, I used x64dbg to set a memory breakpoint on ``unittest_ValidateUnlockCode``, and then proceeded to go through the activation process on Puzzleball 3D's launcher menu.

### Hypothesis
If the memory breakpoint in x64dbg is triggered, then we could revisit the function with a different approach. Otherwise, we would need to trace the user input to the function that **actually performs the activation** or validation step.

## x64dbg Testing
Setting a memory breakpoint in x64dbg can be done through the "Symbols" tab. This is where all of the modules (other libraries needed for app's functionality) are listed along with their imported and exported "symbols" which include variables, data structures, and functions.

<img width="1280" height="720" alt="x64dbg symbols" src="https://github.com/user-attachments/assets/d8a44470-7d96-4bf5-b13d-35d4b1f95dae" />

As we've established earlier, ``ra.dll`` is the binary where the protection mechanism is likely located and is where the ``unittest_ValidateUnlockCode`` function is found under the Symbols tab as expected. To set a breakpoint here, we simply right click on the target function and select "Toggle Breakpoint" or press the F2 shortcut key.

This breakpoint will then be listed under the "Breakpoints" tab in x64dbg with a "hit count" next to it to indicate the number of times the breakpoint was triggered during app runtime.

Now, all that's left to do is to carry out a test to see if the hypothesis is true or false. I navigated to the sub-menu shown by the Puzzleball 3D launcher after clicking the ``Already Paid`` button, and then clicked on the hyperlink to bring me back to the window where the product key was shown along with an input field for the unlock code.

<img width="1280" height="720" alt="not connected" src="https://github.com/user-attachments/assets/3679889d-e5f6-488c-bb22-a96da99c0c9a" />

I entered a random string of characters and clicked the ``SUBMIT`` button which then spawned a pop-up window telling me that I've entered an "unrecognized unlock code".

<img width="1280" height="720" alt="Unrecognized Code" src="https://github.com/user-attachments/assets/600f91fa-bf4d-460c-a0c8-37288203732d" />


