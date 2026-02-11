## Table of Contents
1. [This README](https://github.com/mm-422/Projects--Legacy-App-Reverse-Engineering/edit/main/README.md)
2. [Environment Setup](https://github.com/mm-422/Projects--Legacy-App-Reverse-Engineering/blob/main/environment-setup.md)
3. [Tooling](https://github.com/mm-422/Projects--Legacy-App-Reverse-Engineering/blob/main/tooling.md)
4. [Analysis](https://github.com/mm-422/Projects--Legacy-App-Reverse-Engineering/tree/main/analysis)
	- [Prologue](https://github.com/mm-422/Projects--Legacy-App-Reverse-Engineering/blob/main/analysis/a.%20Prologue.md)
	- [Part 1 - Activation Mechanism](https://github.com/mm-422/Projects--Legacy-App-Reverse-Engineering/blob/main/analysis/b.%20Part%201%20-%20Activation%20Mechanism.md)
	- [Part 2 - Tracing User Input](https://github.com/mm-422/Projects--Legacy-App-Reverse-Engineering/blob/main/analysis/c.%20Part%202%20-%20Tracing%20User%20Input.md)
	- [Continued 1 - Bypassing Integrity Checks](https://github.com/mm-422/Projects--Legacy-App-Reverse-Engineering/blob/main/analysis/d.%20Continued%201%20-%20Bypassing%20Integrity%20Checks.md)
	- [Continued 2 - Decoding Arcade.dat](https://github.com/mm-422/Projects--Legacy-App-Reverse-Engineering/blob/main/analysis/e.%20Continued%202%20-%20Decoding%20Arcade.dat.md)
	- [Part 3 - The Story](https://github.com/mm-422/Projects--Legacy-App-Reverse-Engineering/blob/main/analysis/f.%20Part%203%20-%20The%20Story.md)
	- [Continued 1 - Locating The Judge](https://github.com/mm-422/Projects--Legacy-App-Reverse-Engineering/blob/main/analysis/g.%20Continued%201%20-%20Locating%20The%20Judge.md)
	- [Continued 2 - Bonus](https://github.com/mm-422/Projects--Legacy-App-Reverse-Engineering/blob/main/analysis/h.%20Continued%202%20-%20Bonus.md)
5. [Conclusion](https://github.com/mm-422/Projects--Legacy-App-Reverse-Engineering/blob/main/conclusions.md)
6. [Appendix w/ Additional Artifacts](https://github.com/mm-422/Projects--Legacy-App-Reverse-Engineering/blob/main/artifacts/notes%26images.md)


# Executive Summary
> This repository details a reverse engineering project featuring a legacy software application from the early 2000s.

## Overview 
This project documents a thorough analysis of an early-2000s era application and its components by way of Reverse Engineering. Aspects such as the authentication mechanism, file integrity checks, UI construction and more are covered.

The goal is to demonstrate the importance of understanding an application's intended design and behavior **before** any application of technical steps and methodologies.

The legacy application in question is a video game that was primarily distributed on Windows circa early to mid 2000s. This game implemented custom routines and file validation mechanisms in order to fight off tampering.

While the original servers and publisher are no longer operational, for the sake of ethics, the exact name of this application and images of its components will be obfuscated as necessary.
Hence, the video game app will be referred to as "Puzzleball 3D" throughout the case study.<br>

## Project Goals
- Analyze the structure and behavior of legacy authentication mechanisms.
- Reverse engineer authentication logic with no source code or documentation.
- Demonstrate the importance of thorough research to provide context prior to forming hypotheses and  testing.<br>

## Overall Plan
- Start with basic reconnaissance to gather preliminary info for later analysis.
- Apply static analysis methods to identify functions and algorithm patterns.
- Move to dynamic analysis to observe application behavior.
- Determine potential vulnerabities and attack vectors.
- Document.

## Scope & Ethical Considerations
- This project focuses on understanding mechanisms and applying methodology.
- This project _**DOES NOT**_ distribute material that would encourage piracy.
- No KeyGen or hacktool creation is demonstrated.
- Any example of weaponization potential is done under an educational lens.
- While the application for this project can be found on sites like the Internet Archive, no form of source code or vendor documentation is available.<br>

## Technical Summary
- Observed application behavior with regard to user input.
- Performed static & dynamic analysis on both the main executable and an auxiliary DLL.
- Investigated file integrity checks within application's binary.
- Identified validation logic and performed bypass.
- Evaluated security weaknesses and weaponization potential.<br>

## Tools Used
- _**Disassemblers**_
	- ``Ghidra``
	- ``x64dbg``
- _**Primary Debugger**_
	- ``WinDbg``
- _**Process Inspection**_
	- ``Procmon``
	- ``Spy++``
- _**Auxiliary Tools**_
    - ``HxD`` for editing hex.
    - ``PE-bear`` for quick string searches.
    - ``Detect-It-Easy`` for app property inspection.
    - ``GIMP``, ``Notepad``, and ``LibreOffice`` for mindmaps and notes.<br>
- All testing was performed in a controlled ``Windows 10 22H2`` VM.<br>

## Takeaways
- Focus should be on understanding the intended design behind an application.
- Most if not all client-side validation is "doomed" with modern analysis and debugging tools.
- Clear documentation helps tremendously with prolonged debugging sessions.
- Design oversights and security mistakes seen in a 20-year old app can still be seen today.<br>

## Disclaimer
This repository is provided for educational and research purposes only.
No responsibility is taken for misuse of the information contained herein.
