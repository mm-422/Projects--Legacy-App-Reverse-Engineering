# Appendix
The notes and images included below are supplementary pieces of information gathered throughout the project to demonstrate evidence of analysis.

## The Scratchboard
<img width="1920" height="1080" alt="1 Scratchboard" src="https://github.com/user-attachments/assets/fbac8f5d-988e-4b76-887d-5941ff81ca2b" />

This is simply a layer in GIMP for me to paste and edit images.<br>
You could think of this canvas as a "digital desk space" where I pull together screenshots for performing experiments and doing comparisons between functions, registers, etc. It allows me to work quickly and also reset to a clean workspace without needing to manage and reopen multiple windows.

## String Compilation
<img width="3840" height="1236" alt="2  RE Strings" src="https://github.com/user-attachments/assets/6ecd6654-83cd-4a97-9f66-8ed2f62742d2" />

This is a collection of images with the most useful strings to look for in any RE project, compiled with help from ChatGPT.

## RE String Search Notes
[RE string search.txt](https://github.com/user-attachments/files/25076284/RE.string.search.txt)

This is a text file with "raw notes" on how to sift through a binary for strings and how to approach finding the "entry point" in an application effectively. Here's a snippet of what is contained in the file:
```
Find the Binary's "backbone" instead of random functions
	Look for:
	- imports like CreateFileA, ReadFile, WriteFile, MessageBoxA, DialogBoxParamA, GetDlgItemText, SetWindowText, recv, send
	- Each import is a clue : "This function must be doing X".
	- Entry points like _start, WinMain, DialogProc, WndProc.
	- Once you've found the event dispatcher, you've found 60% of the code's control flow.
```

## Workflow Map
<img width="1981" height="1912" alt="RAdll analysis" src="https://github.com/user-attachments/assets/01793f58-35e3-4323-9910-aea0937ea669" />

This is a snippet of a map of functions in ``unittest_ValidateUnlockCode`` made in GIMP.
It visualizes the flow clearly and shows how the routines relate to one another. Compiling something like this can be extremely time consuming but allows for easy reference and tracking of context when returning to a section of code after a period of time.
	
