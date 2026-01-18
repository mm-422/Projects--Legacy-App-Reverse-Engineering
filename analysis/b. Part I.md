# Part I
> GOAL: Decode the activation mechanism.

I began the analysis by first making simple observations of the application's features. While the initial goal was to decode the activation mechanism, I still had to establish whether that was actually feasible with Puzzleball 3D. Below are the findings:

## Initial Observations
### First Impressions
Launching Puzzleball 3D from either double-clicking the main .EXE or quick start icon presents a stylized "launcher" type menu. The revision number is interestingly shown here ➠ Rev. 3744 Build 177.
This was likely used for internal troubleshooting or to even help with customer support and does not appear to be relevant to the goal, considering the fact that this is a closed-source application that likely did not have a public repository with other versions made available.
- [LAUNCHER IMG]
- [LAUNCHER BUY NOW TEXT IMG]<br>

This text indicates that the game install is "complete" in where all the required assets, data, and features are already present on the system and that it is likely only locked digitally. This is promising because it means reverse-engineering is feasible.

### Exploring The Menus
- [BUTTONS ZOOMED IMG]<br>

Clicking on the "Other Games" button leads to an insecure and likely expired URL. Since the publisher's site is no longer functional (non-existent backend), exploits like SSRF and IDOR are not possible. They would require explicit permission from the server owners anyway and is something outside the scope of this project. But it is interesting to imagine how "fishing" for unlock codes stored in a database could've been a possibility.
- [INSECURE SITE IMG]<br>

Clicking on "Already Paid" brings forth a new window into view with 4 separate tabs.
The text here indicates that an internet connection is required to activate the game or unlock the full version. We could intercept the application's attempt at "calling back to base" and host our own server to spoof the validation process but there is far too little information (protocol, request format, etc.) at this point to go down this route.
- [ALREADY PAID MENU IMG]<br>

Clicking on "I'm not connected to the internet" renders a new screen displaying a URL and interestingly, a product code. This code doesn't seem to change with multiple restarts of the application so it is likely tied to some system property or attribute.
We'll note this down for future reference.
- [NOT CONNECTED MENU IMG]<br>

Examining the URL, we see that the product code is exposed in the parameter, "pid", which likely stands for product ID. It is difficult to ascertain what "did" is.
We'll note down this URL for reference as well.

The other tabs are simply different payment methods for what used to be the way to obtain legitimate unlock/activation codes for Puzzleball 3D. They all require a response from the payment processor's server which is well outside the scope of this project.

## Static Analysis
After having established that reverse-engineering may be possible with Puzzleball 3D, I performed some basic static analysis using PE-bear to sift through string references and Ghidra to provide a graphical view of the functions.
### PE-bear Findings
These are some of the strings in the app's binary that stuck out as potential leads:
- GlobalUnlock @ offset c6fde<br>
- radll_GetUnlockCode @ offset c7280<br>
- unittest_DecryptUnlockCode @ offset c73df<br>
- unittest_ValidateUnlockCode @ offset c7553<br>
- Action Fail Wrong Unlock Code @ offset c86f4<br>
- Form Edit Control Containing Unlock Code @ offset c86f4<br>
- UnlockCode @ offset c8728<br>

There were of course, no hardcoded unlock codes found in the Puzzleball 3D's binary. But "unittest_ValidateUnlockCode" seemed like the most interesting place to start. Looking this up in Ghidra led to a function with that exact name.<br>

### Ghidra Findings
- [VALIDATE UNLOCK UNITTEST FUNCTION IMG]<br>

The above is a "Ghidra representation" of the function, translated from machine code into human-readable form. It seems that there are 3 variables being declared:
- cVar1, a single character data type.
- local_c and local_8, both integer data types.<br>

The routine seems to begin with some processing applied to the arguments supplied (*param_1 and *param_2) along with the aforementioned variables.

A new value for cVar1 is derived from the processing involving function FUN_100180d1 before being compared against '\0' at the end which is likely a null byte.

To understand this routine better, I switched to the assembly view for unittest_ValidateUnlockCode in Ghidra to examine the "assembly logic"<br>
- [VALIDATE UNLOCK UNITTEST FUNCTION IMG]<br>

Beginning from the top, we see that the ECX register's value is pushed twice onto the stack in place of where the ESP register's value would typically be subtracted to "make space". This could just be a form of compiler optimization.

We also see that the function, FUN_1007ccbd, is applied, if you will, in a consistent manner to both param_1 and param_2. The preceding and following instructions to the function call are very similar. Another function towards the end, FUN_1007cdca, also seems to be applied in this way. It could be reasoned that the former is some sort of "preparation step" and the latter is a "finishing step".

These two functions are likely not where the core logic or math for the activation mechanism is located but I still thought it important to delve deeper into FUN_1007ccbd in an attempt to fully deconstruct the routine.<br>

#### FUNCTION 1007CCBD
- [FUN 1007CCBD IMG]

Here we see a nested set of blocks that divide the assembly into several sections. Attempting to decode this part of the parent function, without any dynamic analysis took an immense amount of time and effort.

Suffice to say, this function essentially allocates system memory to load the variables and arguments in an "expected state" before being processed by the other routines found in unittest_ValidateUnlockCode, namely FUN_10018172 and FUN_100180d1.

The presence of standard C library functions like _strlen, _strncpy, as well as operator_new (found within FUN_1007ebe0 in FUN_1007ccbd) are the biggest giveaways.

#### FUNCTION 10018172
Backing out of FUN_1007ccbd and moving onto FUN_10018172, we see a routine that is even more convoluted:<br>
- [FUN 10018172 IMG]

Attempting to decode this function ended up being a mammoth task, largely due to unknown items like the values for "DAT_100dc868" and "DAT_100dc86c" as well as the numerous amount of nested functions.

All that I could ascertain from this summarized view of FUN_10018172 is that if the argument param_2 contains a character 'F', then further processing is applied. Otherwise, the app's workflow would skip to the end and proceed to the next routine in the parent function (unittest_ValidateUnlockCode) which would be FUN_100180d1.

## Making A Decision
Reverse-engineering is a process that can get quickly overwhelming if the right tools and methodology aren't applied. But as we'll see later, even that may not be enough to fully deconstruct something like Puzzleball 3D which is a minimally obfuscated yet ancient application with peculiar design philosophies.

After having spent countless hours trying to uncover the purpose of FUN_100180d1 with no leads, I deemed it a "failure". This however, did not stop me from stepping back and taking an entirely different approach, as will be detailed in Part 2.
