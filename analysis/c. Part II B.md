# Part II B
The complications I encountered throughout trying to decode Puzzleball 3D with its relatively opaque routines and unconventional design led me to take a few steps back before re-evaluating my approach.

I decided to pick a simpler part of the application to modify this time in order to **gather more information** on how it responds to analysis and debugging tools. After the previous failures, I wanted to eliminate any possible compatibility issues and ensure there were no "invisible" obfuscation steps that made using tools more challenging than necessary.

> GOAL: Fix typo on launcher window.<br>

To go about this, I wanted to remedy a misspelling of the word "computer" on the launcher window where the product key was displayed.

<img width="631" height="88" alt="typo" src="https://github.com/user-attachments/assets/2dfb1fc2-3a78-4aeb-8225-25d73ff8f6f9" />

The word "compter" here is missing a vowel, which means we'd have to add an extra byte somewhere in the program's binary or even an external resource file in order to fix the word.

Performing a quick string search through the main .EXE and even the ``ra.dll`` file for the word "compter" returned nothing. It was very unlikely that the application had a function to "construct" words and sentences procedurally, especially on a custom-drawn window. Moreover, the relatively low text quality was also a giveaway; natively rendered text wouldn't exhibit as much as aliasing as can be clearly seen in the screenshot images.

So I looked through the root directory of Puzzleball 3D and came across a file called ``Arcade.dat``. This file was of a non-standard ".dat" type. To quickly inspect its contents, I attempted to open it with Notepad which presented what looked like a header with unintelligible characters, followed by readable text. Using the "find" tool, I was able to locate the misspelled word.

<img width="1280" height="720" alt="arcade notepad" src="https://github.com/user-attachments/assets/f6d89b1a-a536-4585-a1af-224681e918f6" />

It should be stated here that reading and modifying data files with arbitrary extensions using a tool like Notepad is ill advised as it could lead to corruption. I knew even at the time that doing so was less than ideal, but it allowed for quickly sifting through the contents of ``Arcade.dat`` in order to locate the desired word.

Changing the word "compter" to "computer" through Notepad seemed trivial, and the launcher even started up as expected. However, when clicking on the ``Already Paid`` button to access the sub-menu where the product key was displayed, we are presented with this error:

<img width="1280" height="720" alt="fatal not found" src="https://github.com/user-attachments/assets/c953b7e0-a388-4db8-8891-a6d615f7f5cf" />

This essentially confirms that some if not all of the elements on the sub-menu draw from the Arcade.dat file. This also adds up with the fact that the contents of that file resemble a framework, perhaps used for constructing the launcher's graphical interface.

What's interesting is that reverting the word change doesn't seem to fix or undo this error. It should also be noted that the title of the error, "Fatal Not Found", alludes to the application keeping track of the .dat resource file in some way, perhaps with an internal ID or hash. Tampering with it with the Notepad edit likely caused this internal ID to change or become corrupted.

The text within that relatively large error dialog box also resembles something used in a development environment for debugging purposes. Not something intended for the end-user.

## Taking A Detour
> GOAL: Trace the File Not Found Error.<br>

At this point in the project, I assumed that a validation mechanism of some sort kept track of ``Arcade.dat`` in order to detect tampering. So I searched through both the main binary and the DLL file with Ghidra for references to the error string "Fatal Not Found". I found a copy of all the text displayed in the previous dialog box in each of the binaries.

<img width="1280" height="720" alt="fatal not found ghidra" src="https://github.com/user-attachments/assets/97939581-3d13-4f5d-beba-ad7728c41235" />

This was an interesting bit of redundancy. But it could have just been remaining data that went uncleaned after development. So to verify which binary (if either) provided the data for the dialog box, I tried changing the string in Ghidra to see if it would show, starting with the one in the main executable.
Doing this caused the Puzzleball 3D application to throw the following error on startup:

<img width="1280" height="720" alt="CRC error" src="https://github.com/user-attachments/assets/da3c25b9-a100-4fd7-bb34-51e73c8526d9" />

This error dialog box indicates a CRC failure, which likely meant that there was a routine checking the integrity of the main executable.

I tried "tracing up" from the string shown in Ghidra bny following the XREF; in this case the string was found in function FUN_00416659. Attempting any sort of assembly modification (ex. flipping a JNZ instruction to JZ) caused the application to throw another error stating that "Game Files Are Corrupt".

This was another string that can be found in both the main executable and the DLL file. I tried tracing upwards from this string as well and found a common function between the two different string references => ``FUN_00401006``

#### Function 00401006
In order to understand how Puzzleball 3D was making decisions with regard to which error dialog to show, I decided to do a breakdown analysis on the common denominator, function ``FUN_00401006``.
The decompiled view shown in Ghidra alone is convoluted. I have broken it down to numbered blocks for easier view:




