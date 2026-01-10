# Revealing The Story 🕮
> A reverse engineering project demonstrating the importance of behavioral analysis.

### Overview ˙⋆✮
This project documents a thorough analysis of an early-2000s era application and its components by way of _**Reverse Engineering.**_ Aspects such as the authentication mechanism, file integrity checks, UI construction and more will be covered.

The goal is to demonstrate how crucial it is to properly understand an _**application's behavior**_ and intended design _**before**_ applying any technical step and/or methodology.

The legacy application in question is a video game that was primarily distributed on Windows circa early to mid 2000s. This game implemented custom routines and file validation mechanisms in order to fight off tampering.

But as we'll see later, once we uncover "the story" of any particular application, its inner workings and even "secrets" will become clear to see. Doubly so for an app that doesn't strictly require _**server-side**_ authentication and/or authorization.

While the original servers and publisher are no longer operational, for the sake of ethics, the exact name of this application and images of its components will be obfuscated as necessary.
Hence, the video game app will be referred to as "Puzzleball 3D".<br><br>

###  ֎ Project Goals
- Analyze the structure and behavior of legacy authentication and integrity mechanisms.
- Reverse engineer authentication logic with no source code or documentation.
- Demonstrate weaponization potential and relevant phase in a typical kill chain framework.<br><br>

###  ֎ Scope & Ethical Considerations
- This project focuses on understanding mechanisms and applying methodology.
- This project _**DOES NOT**_ distribute material that could encourage piracy.
- The app for this project is available on the Internet Archive and other "abandonware sites".
- No KeyGen or hacktool creation is demonstrated.
- Any examples of weaponization potential is done under an educational lens.<br>
<sup>_They won't really work in modern secured systems anyway!_</sup><br><br>

###  ֎ Technical Summary
- Observed application behavior with regard to user input.
- Performed static & dynamic analysis on both the main .EXE and an auxiliary DLL.
- Investigated file integrity checks within app's code.
- Identified validation logic and performed bypass.
- Evaluated security weaknesses and weaponization potential.<br><br>

###  ֎ Tools Used
- Disassembler ➠ Ghidra & x64dbg
- Primary Debugger ➠ WinDbg
- Process Inspection ➠ Procmon & Spy++
- Auxiliary Tools
    - HxD for editing hex.
    - PE-bear for quick string searches.
    - Detect-It-Easy for app properties.<br>
- _All testing was performed in a controlled Windows 10 22H2 VM._<br><br>

###  ֎ My Takeaways
- Most if not all client-side validation is "doomed" with modern analysis and debugging tools.
- Reverse engineering should be about understanding behavior and intent first.
- Clear documentation helps tremendously with prolonged debugging sessions.
- Design oversights and security mistakes seen in a 20-year old app can still be seen today.<br><br>

###  ֎ Disclaimer
This repository is provided for educational and research purposes only.
No responsibility is taken for misuse of the information contained herein.
