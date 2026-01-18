# Part II
One crucial step I missed in Part I was to verify if the function, "unittest_ValidateUnlockCode", was even a relevant part of Puzzleball 3D's binary used during any actual code validation process ie. inputting unlock code and clicking submit.

If we looked closer to examine the name of the function alone, the word "unittest" here is a hint. 
In software programming, "unit testing" is a method for verifying if individual components of an application are working as expected. In this case, the "unittest_ValidateUnlockCode" function block is likely tied to ensuring that user input validation (like an unlock key) executes as expected.

This does not necessarily mean that the same function is also used for validating "real" user input from the main application. Often times, it is good practice to design isolated code blocks that can be automated for unit testing.

But as we will see later, this isn't always the case with Puzzleball 3D.<br><br>

## New Approach
> GOAL: Verify validity of unittest_ValidateUnlockCode.

Determining the relevancy of "unittest_ValidateUnlockCode" is not something that could be done through static analysis alone and typically requires "stepping through code" with a debugging tool. To do this, I used x64dbg to set a memory breakpoint on "unittest_ValidateUnlockCode", and then proceeded to go through the game activation process.

If the memory breakpoint is triggered, then we could revisit the function with a different approach. Otherwise, we would need to follow the user input to the function or routine that actually performs the activation or validation step.

## x64dbg Testing
Setting a breakpoint in x64dbg can be done by first navigating to the 
