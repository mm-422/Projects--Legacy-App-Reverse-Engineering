# Part I
> GOAL: Decode the activation mechanism.

I began the analysis by first making simple observations of the application's features. While the goal was initially set to decode the activation mechanism, I still had to establish whether that was actually feasible with Puzzleball 3D. Below are the findings:

## Initial Observations
Launching Puzzleball 3D from either double-clicking the main .EXE or quick start icon presents a stylized "launcher" type menu. The revision number is interestingly shown here ➠ Rev. 3744 Build 177.
This was likely only used for troubleshooting on the customer support end and not something relevant to the goal seeing how this is a closed-source application that likely did not have a public repo with other versions made available.
- [LAUNCHER IMG]
- [LAUNCHER BUY NOW TEXT IMG]<br>

This text indicates that the game install is "complete" in where all the required assets and data are already present on the system and that it is likely only locked digitally. This is promising because it means reverse-engineering is feasible.
- [BUTTONS ZOOMED IMG]<br>

Let's take a look at what some of the buttons do.
Clicking on "Other Games" leads to an insecure and likely expired URL. Since the site is no longer functional (non-existent backend), exploits like SSRF and IDOR are not possible. They would require explicit permission from the server owners anyway and is something outside the scope of this project. But it is interesting to imagine that "fishing" for unlock codes stored in a database could've been a possibility.
- [INSECURE SITE IMG]<br>

Clicking on "Already Paid" brings forth a new window into view with 4 separate tabs.
The text here indicates that an internet connection is required to activate the game or unlock the full version. We could intercept the application's attempt at "calling back to base" and host our own server but there is far too little information at this point to go down this route.
- [ALREADY PAID MENU IMG]<br>

Clicking on "I'm not connected to the internet" renders a new screen displaying a URL and interestingly, a product code. This code doesn't seem to change with multiple restarts of the application so it is likely tied to some system property or attribute. We'll note this down for future reference.
- [NOT CONNECTED MENU IMG]<br>

Examining the URL, we see that the product code is exposed in the parameter, "pid", which likely stands for product ID. It is difficult to ascertain what "did" is. We will note down this URL for reference.

The other tabs are simply different payment methods for what used to be the way to obtain legitimate unlock/activation codes. They all require a response from the payment processor's server which is well outside the scope of this project.

## Static Analysis
After having established that reverse-engineering is possible with Puzzleball 3D, I performed some basic static analysis using PE-bear to sift through string references and Ghidra to provide a graphical view of some of the functions.

Some of the strings found through PE-bear stuck out as potential leads:<br>
- GlobalUnlock @ offset c6fde<br>
- radll_GetUnlockCode @ offset c7280<br>
- unittest_DecryptUnlockCode @ offset c73df<br>
- unittest_ValidateUnlockCode @ offset c7553<br>
- Action Fail Wrong Unlock Code @ offset c86f4<br>
- Form Edit Control Containing Unlock Code @ offset c86f4<br>
- UnlockCode @ offset c8728<br>

There were of course, no hardcoded unlock codes found in the Puzzleball 3D's binary. But "unittest_ValidateUnlockCode" seemed like the most interesting place to start. Looking this up in Ghidra led to a function with that exact name.<br>
[VALIDATE UNLOCK UNITTEST FUNCTION IMG]

The above is a "Ghidra representation" of the function, translated from machine code into a human-readable form. It seems that there are 3 variables being declared:
- cVar1, a single character data type.
- local_c and local_8, both integer data types.<br>

The routine seems to begin with some processing applied to the arguments supplied (*param_1 and *param_2) together with the aforementioned variables.

A new value for cVar1 is derived from the processing involving the function FUN_1000180d1 before being compared against '\0' at the end which is likely a null byte.

To understand this routine better, I went a layer deeper to observe the assembly view as shown in Ghidra:<br>
[VALIDATE UNLOCK UNITTEST FUNCTION IMG]
