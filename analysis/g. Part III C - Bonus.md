# Part III C
With the main goal achieved, I wanted to revisit the "typo problem" to tie up loose ends. The same approach was applied to this problem as with decoding the activation mechanism; gaining good understanding of the application's behavior with context from historical research first before applying techniques and methodology.

> GOAL: Fix the typo in Arcade.dat

## General Anti-Tamper Mechanisms
In the early 2000s, developers utilized a few common tricks to ensure the data files weren't tampered with. Some of these are as follows:
- **The Checksum / CRC Trap**<br>
```
- App reads every byte in the data file.
- Adds them up or runs an algorithm like CRC32.
- Compares the result to a hardcoded value.
```
- **The File Size Trap<br>**
```
- App asks Windows, "How big is data file X?"
- If reported size =/= internally expected size, then app assumes the file is corrupt.
- Ex. Changing "compter" to "computer" in Arcade.dat required adding an extra byte.
```
- **The Offset/Pointer Table Trap**<br>
```
- Custom UI files often have a header at the start.
- This header states the starting point of elements like strings.
- Modifying the data file by adding an extra byte shifts the entire body forward.
- This causes a mismatch between the starting point in the header and the body.
```
♦️ Testing Checksum Logic
An easy test to determine if there exists a CRC check for an external resource file like ``Arcade.dat`` is to modify the contents while keeping the **file size the exact same.**

We simply open ``Arcade.dat`` in a hex editor like HxD and perform a byte change while keeping the total number of characters the same. Example ➜ changing the word "compter" to "computr". If the app works as expected then we are likely not dealing with a checksum or CRC check.

Below is a snippet of documentation from the preliminary tests performed on ``Arcade.dat``.

<img width="814" height="241" alt="crc test" src="https://github.com/user-attachments/assets/cb0d0c9a-1106-498c-a3a9-21e42c7d198d" />

## Magic Bytes & Headers
Earlier in the project, I mentioned performing a quick edit to ``Arcade.dat`` with Notepad, a simple text editor. This was of course not ideal as corruption may occur from improperly modifying the contents of a file without first establishing its structure and format.

There are other ways to verify a file's type aside from examining the extension which can be arbitrarily set. With HxD, we can quickly view the first few bytes of resource files like ``Arcade.dat`` to uncover important clues.

<img width="1249" height="377" alt="magicbytes" src="https://github.com/user-attachments/assets/7be9d7b1-fd63-4371-958f-cbc95a316a7c" />

The first four bytes, ``50 4B 03 04`` are the magic bytes for a ZIP archive, and this makes sense.

It was incredibly common for developers in the early 2000s to bundle resources and assets for video games into a standard ZIP file, and then simply rename the extension to something like ".dat", just like how Java uses ".jar" for example.

##

Simply editing ``Arcade.dat`` with a text editor didn't just change the bytes, it corrupted the internal structure of the ZIP

