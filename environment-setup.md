# The Environment
A testing environment including all the necessary tools and appropriate settings is paramount to reverse engineering, especially when analyzing (potentially) malicious software.

I prefer using a virtual machine for these tasks as they are easy to spool up with the exact configurations required. The "running state" can also be saved with a click of a button; **invaluable for long debugging sessions.**

Below are the environment details along with the specific settings applied.

## The Virtual Machine
- Set up Windows 10 22H2 64-bit through VirtualBox version 7.2.4.r170995.
- Puzzleball 3D was likely designed for Windows XP 32-bit.
- However, replicating the exact OS env is not necessary for the goals of this project.
<img width="1280" height="720" alt="VM DESKTOP" src="https://github.com/user-attachments/assets/78035289-022b-4aff-8589-c7b2672860eb" />

## Resources
- Allocated 4 logical cores (Zen 3 CPU) and 8GB of memory to Virtual Machine.
- This was sufficient for up to 3 instances of Ghidra + 1 instance of WinDbg simultaneously.

## Files & File Paths
- Created a backup of Puzzleball 3D's files on external media.
- Installed Puzzleball 3D in default directory under "Program Files (x86)".
- Duplicated Puzzleball 3D's root directory and files to Desktop.
- All testing and modifications to the app's binaries was performed on the files here.

## OS-level Settings
- Disabled network connection.
- Added the "Desktop" directory to Windows Defender's exception list.
- Left ASLR and default memory integrity settings like Core Isolation alone.
