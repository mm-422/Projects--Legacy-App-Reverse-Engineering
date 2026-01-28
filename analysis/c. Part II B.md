# Part II B
The complications I encountered throughout trying to decode Puzzleball 3D with its relatively opaque routines and unconventional design led me to take a few steps back to re-evaluate my approach.

I decided to pick a simpler part of the application to modify this time in order to **gather more information** on how it responds to analysis and debugging tools. After the previous failures, I wanted to eliminate any possible compatibility issues and ensure there were no "invisible" obfuscation steps that made using tools more challenging than necessary.

> GOAL: Fix typo on launcher window.<br>

To go about this, I wanted to remedy a misspelling of the word "computer" on the launcher window where the product key was displayed.

<img width="631" height="88" alt="typo" src="https://github.com/user-attachments/assets/2dfb1fc2-3a78-4aeb-8225-25d73ff8f6f9" />

The word "compter" here is missing a vowel, which means we'd have to add an extra byte somewhere in the program's binary or maybe even an external resource file in order to correct the word.

Performing a quick string search through the main .EXE and even the ``ra.dll`` file for the word "compter" returned nothing. It was very unlikely that the application had a function to "construct" words and sentences procedurally, especially on a custom-drawn window. Moreover, the relatively low text quality was also a giveaway; natively rendered text wouldn't exhibit as much aliasing as can be clearly seen in the screenshot images.

So I looked through the root directory of Puzzleball 3D and came across a file called ``Arcade.dat``. This file was of a non-standard "DAT" type. To quickly inspect its contents, I attempted to open it with **Notepad** which presented what looked like a header with unintelligible characters, followed by readable text. Using the "find" tool, I was able to locate the misspelled word.

<img width="1280" height="720" alt="arcade notepad" src="https://github.com/user-attachments/assets/f6d89b1a-a536-4585-a1af-224681e918f6" />

It should be stated here that reading and modifying files with arbitrary extensions using a tool like Notepad is ill advised as it could lead to corruption. I knew even at the time that doing so was less than ideal, but it allowed for quickly sifting through the contents of ``Arcade.dat`` in order to locate the desired word.

Changing the word "compter" to "computer" through Notepad seemed trivial, and the launcher even started up as expected. However, clicking on the ``Already Paid`` button to access the sub-menu where the product key was displayed presented this error:

<img width="1280" height="720" alt="fatal not found" src="https://github.com/user-attachments/assets/c953b7e0-a388-4db8-8891-a6d615f7f5cf" />

This essentially confirms that some if not all of the elements on the sub-menu draw from the Arcade.dat file. This also adds up with the fact that the contents of that file resemble a framework, perhaps used for constructing the launcher's graphical interface.

What's interesting is that reverting the word change doesn't seem to fix or undo this error. It should also be noted that the title of the error, "Fatal Not Found", alludes to the application keeping track of the .dat resource file in some way, perhaps with an internal ID or hash. Tampering with it with the Notepad edit likely caused this internal ID to change or become corrupted.

The text within that relatively large error dialog box also resembles "verbose language" that would be used in a development environment for debugging purposes. Certainly not something intended for the end-user.

## Taking A Detour
> GOAL: Trace the "File Not Found" error.<br>

At this point in the project, I assumed that a validation mechanism of some sort kept track of Arcade.dat in order to detect tampering. So I searched through both the main .EXE and ra.dll with Ghidra for any references to the error string "Fatal Not Found".

I ended up finding a row of defined strings in both binaries that contained all the text shown in the previous error dialog box.

<img width="1280" height="720" alt="fatal not found ghidra" src="https://github.com/user-attachments/assets/97939581-3d13-4f5d-beba-ad7728c41235" />

This was an interesting bit of redundancy. The text duplication could just be residual data that survived post-development, in which case it would be next to meaningless for our goal.

To verify if this was the case, and to see which binary (main .EXE or ``ra.dll``, if either) provided the data for the "Fatal Not Found" dialog box, I made a slight change to that string reference in Ghidra (starting with the main .EXE) to see if it would show the next time the error dialog was drawn.

However, doing this caused Puzzleball 3D to throw the following error message on startup:

<img width="1280" height="720" alt="CRC error" src="https://github.com/user-attachments/assets/da3c25b9-a100-4fd7-bb34-51e73c8526d9" />

This error indicates a CRC (Cyclic Redundancy Check) failure of some sort, which might point to the existence of a routine in the main .EXE that verifies its integrity.

I tried following this "CRC failed" string through the XREF shown in Ghidra and landed on the function ``FUN_0040283b``. Attempting any sort of modification to the assembly here, or anywhere for that matter, caused the application to throw another error dialog stating that "Game Files Are Corrupt".

Tracing this new error through the main .EXE brought me to function that seemed to be a common denominator between the two errors => ``FUN_00401006``

#### FUNCTION 00401006
<IMG>
In order to understand how Puzzleball 3D was making decisions with regard to which error dialog to show, I decided to perform a full breakdown of ``FUN_00401006``.

The fact that this routine possessed numerous XREFs pointing to it hinted at it being a global initializer.

Both LAB_0040102b and LAB_00401031 appeared to load data segments into registers before either returning or jumping to a different section as if prepping the app. They are likely responsible for loading resource files like Arcade.dat and populating internal structures.

The first prominent function in this entire routine seems to be FUN_00401d0b which eventually leads us to the location of the error strings.

#### FUNCTION 00401d0b
This is a large function block with many sub-routines. The following is a compiled summary of the assembly code.




