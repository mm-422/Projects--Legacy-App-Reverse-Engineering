# Part III A
In the previous part, we saw how methodology alone could become insufficient when put up against a complex routine with little to no context. What worked well for uncovering the earlier validation mechanisms, became quickly diminished in the face of a relatively opaque function that made even brute-forcing unfeasible.

We should remember that **no tool or technique** in Reverse Engineering is a _one-size-fits-all_ type of solution. They are simply a means to an end, which is to piece together the original developer's intent behind the design of an application.

This "pieced-together intent" is what I like to call the ``Story``.<br>
Essentially, this is the expected flow of an application and/or its components from start to end. It is not just about **what** the application does, but **how** it does something.

Gaining a solid understanding of an application's ``Story`` will allow us to be more measured and accurate when analyzing binaries as well as reduce the likelihood of falling into "self-made traps" that might stem from bad assumptions. 

## The Activation Mechanism Story
For local applications like Puzzleball 3D that don't strictly require any server-side verification, the ``Story`` for its activation/validation mechanism might be as simple as gathering an input, applying some math or processing, and then producing an output based on the result.

Since all the resources needed for the proper and complete functionality of the application is contained within its binary and the system it is hosted or installed on, it should only be a matter of **time, effort, and most importantly understanding,** before every detail is unobfuscated and every "secret" unraveled.

## Gathering Historical Context
In order to flesh out the ``Story`` for Puzzleball 3D's activation mechanism, we first need to get into the mind of the original developer(s). This requires some research on period-relevant information, hence the "historical context".

### ♦️ Early Application Design
Back in the 2000s, the ``Story`` of a particular application usually revolved around a specific and procedural path, especially with regard to its activation/validation system.

At it's core, the system would begin with gathering an input, applying some logic and/or processing to that input, and then constructing an appropriate output based on the result.

For applications that relied on external modules for validation routines ― like a proprietary DLL in the case of Puzzleball 3D ― there would exist an extra "hand off" phase or "bridge" in between the input and the logic block.

This flow can be visualized as follows:
```
User Input ➜ Bridge ➜ DLL ➜ Logic/Processing ➜ Output
```
Essentially, the application's main executable would act like a **messenger**, collecting the user input ― like through a prompt, button click, and so on ― before passing it to the associated DLL with the help of a "bridge" function.

The DLL then runs that input through some processing where a "Judge" function would ultimately decide whether it is valid. Finally, the DLL will construct an output based on the result of the processing.

### ♦️ What is the Judge?
``The Judge`` is simply a routine that determines if an input is valid or matches an expected output.

Since the logic for Puzzleball 3D's activation mechanism is likely located in ``ra.dll``, and the launcher is custom-built, there must exist an intermediate function or "bridge", that helps pass the collected input ― which would be the unlock code in this case ― to the DLL in order for the ``Judge`` to run it through the "meat grinder" so to speak.

If we could locate the ``Judge`` and modify it to return a desired result **after** the processing ― like a 1 instead of 0 ― we could bypass the activation mechanism.

But before that, we should take a look at the custom-drawn UI for Puzzleball 3D to see how it relates to the ``Judge`` and how it might affect other parts of the application's design.

### ♦️ Custom Render Engines
It was not uncommon for applications in the early 2000s to implement their own "mini rendering engine" for constructing custom interfaces. This was done to make the apps stand out and bypasses the standard Windows controls in order to provide benefits like a potentially improved user experience. A great example of this is the popular media player (at least back then) called Winamp.

When we throw this "mini rendering engine" into the mix, the ``Story`` of an application's validation mechanism changes slightly.

Instead of the DLL simply constructing and reporting the output at the end, it will attempt to communicate with the function that is actually responsible for constructing the appropriate output based on an "internal ID" system.

Let's say the result after processing an unlock code is ``Unrecognized Code`` instead of ``Wrong Code`` ― both being different but essentially meaning "invalid code" ― which carries a specific internal ID of ``101``. This ID will then be passed from the DLL, through possibly another "bridge", over to the function responsible for constructing a message tied to that ID.

<img width="1280" height="720" alt="Unrecognized Code" src="https://github.com/user-attachments/assets/0f9be2cf-25a7-44e6-80b1-912e667e462b" />

A quick sift through the ``Arcade.dat`` file with Notepad again, revealed that most if not all of the text found on this part of Puzzleball 3D's launcher ― like the word "compter" and "Unrecognized Unlock Code" ― is pulled from that specific resource file.

There must exist a routine in either the main .EXE or ``ra.dll`` that will act as a "Librarian" to grab the correct error string to display based on the internal ID system mentioned previously.

### ♦️ What is the Librarian?
This is usually a function responsible for loading items/resources into memory and then constructing a "map" to keep track of them. It then retrieves the required item straight from a location in memory through the use of pointers or IDs.

Why load those items into memory instead of pulling them straight from the disk?
This is largely due to hard drives being prolific in the 2000s. This mainstream storage medium was much slower than system memory. If an application had to call the ``Librarian`` to fetch a specific resource each time a user interaction ― like a button click ― was performed, the UI would incur stutters and lead to poor user experience.

This is a critical bit of understanding that helps shed light on the flow of Puzzleball 3D's routines and components while aiding in making decisions like knowing when to search for data through the system memory instead of the local storage device.  

### ♦️ Tracking the Librarian
Uncovering the location of the ``Librarian`` could come in handy later when attempting to say, trace error messages up to the ``Judge`` through the use of pointers or IDs in memory.

Instead of setting a breakpoint on string references, we could consider placing them on calls to Windows APIs that allow the application to communicate with external data/resource files like ``Arcade.dat``.

Since error strings like ``Unrecognized Unlock Code`` are found in ``Arcade.dat`` ― which is an external file ― it would be more practical to set a breakpoint on API calls like ``CreateFile`` and ``ReadFile``, which are more than likely used to open ``Arcade.dat`` and read from it before pulling the required string.

We can trace our way to the ``Librarian`` by:
1. Loading Puzzleball 3D on WinDbg.
2. Navigating to the input field in the launcher's sub menu.
3. Setting a breakpoint through WinDbg with the command ➜ ``bp kernel32!ReadFile``
4. Entering a fake code to initiate the process of drawing the error dialog.
5. Looking at the call stack when the breakpoint hits.
6. Finding the parent class ➜ This is the ``Librarian``.

It is important to note that an application's ``Librarian`` is usually responsible for handling requests from multiple sources and is almost never tied to just loading a single file; otherwise the ``ReadFile`` API alone would be sufficient.

Below is what the call stack in WinDbg looked like after the breakpoint on ``ReadFile`` was hit:

<img width="697" height="317" alt="librarian" src="https://github.com/user-attachments/assets/370d4fe7-d608-4563-9fe8-d6c57529dfc6" />

It was still unclear which exact routine was the "core" ``Librarian`` and which ones were simply child functions. This is why I placed temporary labels on the image in order to use as a reference later on.

Since Puzzleball 3D relies on more than one external file ― like ``RAW_003.wdt``, ``Arcade.dat``, ``Channel.dat`` and so on ― the breakpoint was hit a total of 137 times subsequently. I had to correlate this data with observations in Procmon in order to narrow down the exact breakpoint that is tied to the loading routine for ``Arcade.dat``.

To do this, I set Procmon up with the following filters in addition to the defaults:

<img width="1027" height="757" alt="procmon filters" src="https://github.com/user-attachments/assets/50c5c570-6318-4c38-9a06-c7d7641d7095" />

I then "stepped the execution forward" in WinDbg while monitoring the Procmon window for events.

<img width="990" height="722" alt="procmon readfile" src="https://github.com/user-attachments/assets/5bc3337d-f5b1-4065-93e5-5d8141360904" />

Upon repeated tests with the same breakpoint location as above, ``Arcade.dat`` is confirmed to load on the fourth ``ReadFile`` operation each and every time. Checking the call stack in WinDbg now shows us the exact stack structure that we should investigate:

<img width="697" height="317" alt="readfile exact" src="https://github.com/user-attachments/assets/993644e0-97ed-4df4-bc7f-429cfaaffb87" />

After a lot of work tracing through the functions, the overall flow for accessing ``Arcade.dat`` is as follows:
```
radll_radll_Initialize
  ➜ ArcadeLoader
    ➜ ArcadeLoaderCORE (3rd instance)
      ➜ FUN_100837c2
        ➜ FUN_10084328
          ➜ FUN_10083f39
            ➜ LBR-4_1008008f (Wrapper. Multiple calls found)
              ➜ LAB_10080bb5 section under FUN_100809ee
                ➜ LBR-3_100a26de
                  ➜ LBR-2_100a270d
                    ➜ LBR-1_100a900a
                      ➜ LBR_ReadFile
```
