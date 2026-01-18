# Revealing The Story 🕮
> A reverse engineering project demonstrating the importance of behavioral analysis.

## Overview 
This project documents a thorough analysis of an early-2000s era application and its components by way of _**Reverse Engineering.**_ Aspects such as the authentication mechanism, file integrity checks, UI construction and more are covered.

The goal is to demonstrate how crucial it is to properly understand an _**application's behavior**_ and intended design _**before**_ applying any technical step and/or methodology.

The legacy application in question is a video game that was primarily distributed on Windows circa early to mid 2000s. This game implemented custom routines and file validation mechanisms in order to fight off tampering.

But as we'll see later, once "the story" of a particular application is uncovered, its inner workings and even "secrets" will become clear to see. Doubly so for an app that doesn't strictly require _**server-side**_ authentication and/or authorization.

While the original servers and publisher are no longer operational, for the sake of ethics, the exact name of this application and images of its components will be obfuscated as necessary.
Hence, the video game app will be referred to as "Puzzleball 3D" throughout the case study.<br>

###  ֎ Project Goals
- Analyze the structure and behavior of legacy authentication mechanisms.
- Reverse engineer authentication logic with no source code or documentation.
- Illustrate weaponization potential and relevant phases in a typical kill chain framework.<br>

###  ֎ Overall Plan
- Start with basic reconaissance to gather context and preliminary info for later analysis.
- Apply static analysis methods to identify functions and algorithm patterns.
- Move to dynamic analysis to observe application behavior.
- Determine potential vulnerabities and attack vectors.
- Document.

###  ֎ Scope & Ethical Considerations
- This project focuses on understanding mechanisms and applying methodology.
- This project _**DOES NOT**_ distribute material that could encourage piracy.
- The app for this project is available on the Internet Archive and other "abandonware sites".
- No KeyGen or hacktool creation is demonstrated.
- Any examples of weaponization potential is done under an educational lens.<br>
<sup>_They won't really work in modern secured systems anyway!_</sup><br>

###  ֎ Technical Summary
- Observed application behavior with regard to user input.
- Performed static & dynamic analysis on both the main .EXE and an auxiliary DLL.
- Investigated file integrity checks within application's binary.
- Identified validation logic and performed bypass.
- Evaluated security weaknesses and weaponization potential.<br>

###  ֎ Tools Used
- Disassembler ➠ Ghidra & x64dbg
- Primary Debugger ➠ WinDbg
- Process Inspection ➠ Procmon & Spy++
- Auxiliary Tools
    - HxD for editing hex.
    - PE-bear for quick string searches.
    - Detect-It-Easy for app property inspection.
    - GIMP, Notepad, and LibreOffice for mindmaps and notes.<br>
- _All testing was performed in a controlled Windows 10 22H2 VM._<br>

###  ֎ Takeaways
- Most if not all client-side validation is "doomed" with modern analysis and debugging tools.
- Reverse engineering should focus on understanding behavior and intent first.
- Clear documentation helps tremendously with prolonged debugging sessions.
- Design oversights and security mistakes seen in a 20-year old app can still be seen today.<br>

### ⚠ Disclaimer
This repository is provided for educational and research purposes only.
No responsibility is taken for misuse of the information contained herein.
