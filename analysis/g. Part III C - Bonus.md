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
### ♦️ Testing Checksum Logic
An easy test to determine if there exists a CRC check for an external resource file like ``Arcade.dat`` is to modify the contents while keeping the **file size the exact same.**

We simply open ``Arcade.dat`` in a hex editor like HxD and perform a byte change while keeping the total number of characters the same. Example ➜ changing the word "compter" to "computr". If the app works as expected then we are likely not dealing with a checksum or CRC check.

Below is a snippet of documentation from the preliminary tests performed on ``Arcade.dat``.

<img width="814" height="241" alt="crc test" src="https://github.com/user-attachments/assets/cb0d0c9a-1106-498c-a3a9-21e42c7d198d" />

## Magic Bytes & Headers
Earlier in the project, I mentioned performing a quick edit to a typo in ``Arcade.dat`` using Notepad, a simple text editor. This was of course not ideal as corruption can occur from improperly modifying the contents of a file without first establishing its structure and format.

There are other ways to verify a file's type aside from examining the extension such as ``.exe`` or ``.dat`` which can be arbitrarily set. With a more appropriate tool like HxD (hex editor), we can quickly view the first few bytes of resource files like ``Arcade.dat`` to uncover important clues.

<img width="1249" height="377" alt="magicbytes" src="https://github.com/user-attachments/assets/7be9d7b1-fd63-4371-958f-cbc95a316a7c" />

The first four bytes, ``50 4B 03 04`` are the magic bytes for a ZIP archive and this makes sense considering the type of application we are working with.

It was incredibly common for developers in the early 2000s to bundle resources and assets for video games into a standard ZIP file, and then simply rename the extension to something like ``.dat``, just like how Java uses ``.jar`` for example.

This following is the displayed file structure of ``Arcade.dat`` in command prompt after decompression.

<img width="412" height="420" alt="arcade files" src="https://github.com/user-attachments/assets/aea4b380-818c-4162-acc6-2190d4995193" />

### ♦️ The ZIP Archive Structure
Just like how a website's html code consists of sections like the header, body, and footer, the structure of a ZIP file has the following parts:
- **Local File Headers**<br>
```
- Each file inside the ZIP archive starts with a header.
- These contain the filename and the compressed/uncompressed size.
- Easy to locate in HxD by locating the file's path in the decoded text view.
```
- **Payload**<br>
```
- The actual data or "body" of the resource file.
- Contains items such as the error strings.
- Usually not very readable through HxD.
```
- **The Central Directory**<br>
```
- This is located at the end of a ZIP file's binary.
- It is a "master map" that contains the exact position of every file in the archive.
- The app looks for files based on the byte offset listed here.
- This is more efficient than scanning through the entire archive for the desired file.
- Position is indicated by the byte sequence ``02 01 4b 50`` or PK\x01\x02 in ASCII.
```
- **The EOCD**<br>
```
- Stands for End of Central Directory.
- Located at the end of the Central Directory.
- Contains a byte offset that points to the start of the Central Directory.
- Indicated by byte sequence ``50 4B 05 06``.
```

### ♦️ Why Arcade.dat Became Corrupted
Based on what we know about the typical structure of a ZIP archive, we can see how simply adding arbitrary bytes with a text editor like Notepad will cause the internal map to get corrupted.

This is because the process of adding bytes will shift the internal offsets forward. This causes a mismatch between where the header of the files are and the byte offsets listed in the Central Directory.

This also tracks with why overwriting a byte, like changing "compter" to "computr", worked without issues. This is because this modification preserves the file header positions and maintains the byte offsets.

### ♦️ Hypothesis




