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

Tracing this new error through the main .EXE brought me to function that seemed to be a common denominator between the two errors ► ``FUN_00401006``

#### FUNCTION 00401006
<img width="1280" height="720" alt="401006 assembly" src="https://github.com/user-attachments/assets/801456be-3a41-4bb9-8dc5-6da7c7403ef9" />


In order to understand how Puzzleball 3D was making decisions with regard to which error dialog to show, I decided to perform a full breakdown of ``FUN_00401006``. The fact that this routine possessed numerous inbound XREFs hinted at it being a global initializer of some sort.

Both ``LAB_0040102b`` and ``LAB_00401031`` appeared to load data segments into registers before either returning or jumping to a different section as if "prepping" the application. They are likely responsible for loading resource files like ``Arcade.dat`` and populating internal structures.

The first prominent function call here seems to be ``FUN_00401d0b`` which is part of the routine chain that eventually leads us to the location of the error strings.

#### FUNCTION 00401D0B
```
int * __fastcall FUN_00401d0b(int *param_1)
{
uint * _Str;
uint *puVar1;
size_t sVar2;
HANDLE hFindFile;
undefined4 uVar3;
int *piVar4;
CHAR local_944 [2048];
_WIN32_FIND_DATAA local_144;

FUN_004027ab((int)param_1);
param_1[0x4c] = 0;
param_1[2] = 0;
param_1[3] = 0;
GetCurrentDirectoryA(0x104,(LPSTR)(param_1 + 0x4d));

FUN_00401e50();
FUN_004025bf((int)param_1);
_Str = (uint *)(param_1 + 9);
FUN_00401e36((LPSTR)_Str,0x104);
puVar1 = FUN_004170f0(_Str,&DAT_0042e510);
while (puVar1 !=  (uint*)0x0){
  puVar1 = FUN_004170f0(_Str,&DAT0042e510);
  FUN_00417890(_Str,(uint*)((int)puVar1 + 1));
  puVar1 = FUN_004170f0(_Str,&DAT_0042e510);
}

puVar1 = FUN_004170f0(_Str,&DAT_0042e50c);
while (puVar1 != (uint*)0x0){
  puVar1 = FUN_004170f0(_Str,&DAT_0042e50c);
  FUN_00417890(_Str,(uint*}((int)puVar1 + 1));
  puVar1 = FUN_004170f0(_Str,&DAT_0042e50c);
}

sVar2 = _strlen((char*)_Str);
FUN_00417890((uint*)(sVar2 + 0x21 + (int)param_1), (uint*)&DAT_0042e508);
hFindFile = FindFirstFileA((LPCSTR)_Str,&local_144);
if (hFindFile == (HANDLE)0xffffffff){
  FUN_00417890(_Str,(uint*)s_RAW_001.exe_0042e4fc);
}

FindClose(hFindFile);
FUN_00401e36(local_944,0x800);
FUN_004027c6(param_1,local_944);
piVar4 = param_1 + 0x4a;
uVar3 = FUN_004027c2((int)param_1);
FUN_00401091(s_RA\RAW_002.wdt_0042e4e0,param_1,uVar3,piVar4);
return param_1;
}
```

The above is the decompiled view of ``FUN_00401D0B`` as presented in Ghidra. Decoding the assembly for this routine was a long and arduos task. So, for the sake of brevity, I will skip the full analysis and list down the most important findings.

At this point of the project, ``FUN_00401D0B`` seemed like an initializer for integrity checks relating to file corruption. It starts by gathering directory and module names and attempts to locate files\binaries like ``RAW_001.exe`` and ``RAW_002.wdt``, the former being an existing file in the root directory of Puzzleball 3D. Interestingly, ``RAW_002.wdt`` was not a file that existed anywhere on the system (root, Documents, Local App Data, etc.) and may have been a "fall back" for if ``RAW_001.exe`` was not found.

The call to function ``FUN_00401091`` which is found under ``FUN_00401DEB`` is likely what leads to the construction of the "Game Files Are Corrupt" error dialog.

By using x64dbg, I was able to determine that with the current modifications done to Puzzleball 3D's binary, the app took the path towards ``FUN_004027C6`` which is presumed to be a wrapper for the "CRC fail" error. This is our next point of examination.

#### FUNCTION 004027C6
```
**FUN_004027C6**
undefined4 __thiscall FUN_004027C6 (void * this, LPCSTR param_1)
assume FS_OFFSET=0xffdff000

undefined4  EAX:4  <RETURN>
void *  ECX:4(auto)  this
LPCSTR  Stack[0x4]:4  param_1

PUSH EDI
MOV EDI,this
CMP dword ptr [EDI+0x4],0x0
JNZ LAB_0040282A
PUSH ESI
CALL FUN_00402830
PUSH dword ptr [ESP+param_1]
PUSH EAX
CALL FUN_0040283B
POP this
MOV ESI,PTR_s_RA\RAW_003.wdt_0042E6C0
POP this
MOV this,dword ptr [PTR_s_RA\RAW_003.wdt_0042E6C0]
JMP LAB_004027FC


**LAB_004027EE**
PUSH this=>s_RA\RAW_003.wdt_0042EB70
PUSH EAX
CALL FUN_0040283B
POP this
ADD ESI,0x4
POP this
MOV this,dword ptr [ESI]=>DAT_0042E6C4


**LAB_004027FC**
TEST this,this
JNZ LAB_004027EE
MOV this,dword ptr [PTR_s_RA\Background.jpg_0042E6C8]
MOV ESI,PTR_s_RA\Background.jpg_0042E6C8
JMP LAB_0040281B


**LAB_0040280D**
PUSH this=>s_RA\Background.jpg_0042EB50
PUSH EAX
CALL FUN_00402913
POP this
ADD ESI,0x4
POP this
MOV this,dword ptr [ESI]=>PTR_s_RA\button_normal.jpg_0042e6cc


**LAB_0049281B**
TEST this,this
JNZ LAB_0040280D
PUSH EAX
CALL FUN_00402834
POP this
MOV dword ptr [EDI+0x4],EAX
POP ESI


**LAB_0040282A**
MOV AL,0x1
POP EDI
RET 0x4
```

This function is quite evidently loading a "list" of items into memory with references to asset files like ``RAW_003.wdt``,``background.jpg`` and ``button_normal.jpg``. Some of these resources are used when drawing the custom launcher window.

Functions ``FUN_0040283B`` and ``FUN_00402913`` are called to perform some processing on those assets before the call to ``FUN_00402834`` which returns a result that is stored in ``[EDI+0x4]``, then moved to the EAX register just before the final RET instruction.

This result may be the value that indicates the expected or computed "CRC state".

### ✦ Hypothesis
Since ``FUN_004027C6`` seems to compute some sort of hash or value based on a "static list" of items and then stores this result in ``EDI+4``, and then in EAX, before it gets passed back to the parent, ``FUN_00401D0B``, theoretically, editing the value of EAX right before the last step, to something matching an original and untampered version of the main executable, should allow us to bypass the "Game File Are Corrupt" check.

## WinDbg Testing
I first needed to observe the expected value in the EAX register at the right moment. To do this, I set a memory breakpoint at address ``00402826``, which was the start of ``FUN_00402834`` (called under ``LAB_0049281B`` as can be seen above) through WinDbg with the command ► ``bp 00402826``

The value contained in the EAX register at this moment was ``5C1D48A2``. I noted this down and relaunched Puzzleball 3D, this time with the "modified" version of the executable that threw out the CRC errors.
<img width="780" height="222" alt="EAx mod" src="https://github.com/user-attachments/assets/3dc13070-5d00-40c6-8fd5-fe83df72f6e7" />

I changed the ``MOV EAX, dword ptr [ESP + param_1]`` line to instead move an explicit value (``5C1D48A2``) into the EAX register with the following instruction ► ``MOV EAX,0x5C1D48A2``

I then "nullified" the following line by changing ``NOT EAX`` to a simple ``NOP`` instruction.

This then returned the expected value in EAX right before the call to function ``FUN_00401091`` in ``FUN_00401D0B``.

After performing this modification, the application launched normally through the modified main .EXE and we are now free to make alterations to that executable without tripping any integrity checks. It was now time to check the launcher's sub-menu again and examine the typo.

## Back On Track
With the CRC and "corruption" checks neutered, we could now load the version of Puzzleball 3D's executable with the edited strings to see if the "Fatal Not Found" error dialog reflected anything different.
<img width="711" height="470" alt="same error" src="https://github.com/user-attachments/assets/83ebc6f6-2742-433d-aeec-99ca69fffdfe" />

Unfortunately, the error dialog seems unchanged. Modifying any of the string elements here that are found in the main .EXE doesn't break the application now but the changes don't seem to be reflected for some reason. I tried restarting the VM in order to eliminate the possibility of caching but that did not change the result.

There was only one possibility for this scenario ► Puzzleball 3D was drawing the text elements from another source. The DLL file,``ra.dll``, was the next likely culprit. It contained an exact copy of all the strings found in the error in almost the exact order in its binary.

<img width="896" height="793" alt="dll fatal not found" src="https://github.com/user-attachments/assets/b6c8e6cb-37cf-45ba-8fb8-f6af92f2dc28" />

Modifying the string here through Ghidra are once again trivial. Simply right-click on the defined string and select "Patch Data", enter the desired set of characters, and then press the "O" key for the shortcut to compile and output the DLL.

However, performing this now caused Puzzleball 3D to throw a new error dialog on startup.
<img width="368" height="129" alt="dll app error" src="https://github.com/user-attachments/assets/fde27c52-dd43-445f-a3e8-892112680302" />

Restoring the original DLL binary allowed the app to launch normally again, which meant the existence of another "check" for the loading routine for ``ra.dll``.

>GOAL: Modify the app to allow for modded DLL.

