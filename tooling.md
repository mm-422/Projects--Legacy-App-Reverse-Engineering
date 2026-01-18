# The Tools
## Summary
Initially, I relied on just x64dbg and PE-bear to do the heavy lifting. But this was only sufficient for some static analysis. Eventually, I moved on to Ghidra + WinDbg as my potent duo, while also discovering plenty of incredible tools like Spy++ and Detect-It-Easy along the way.

I would also be remiss to not mention the incredible use I've found in leveraging LLMs like ChatGPT and Gemini in order to perform tasks such as research for application design philosophies relevant to the early 2000s era as well as sifting through hundreds of lines of assembly code.

Care should be taken when utilizing these technologies, especially when sensitive info is concerned. Considering the fact that Puzzleball 3D is openly available on sites like the Internet Archive with no relevant owners to contact and that hosting a local LLM would present its own set of challenges (cost, speed, reliability), the use of online hosted LLMs is justified in my opinion.

## Static Analysis
- PE-bear <sub>v0.7.1</sub>
- Ghidra <sub>v11.4.2</sub>

## Dynamic Analysis
- WinDbg <sub>v1.2511.21001.0</sub>
- x64dbg <sub>v0.0.2.5</sub>
- Procmon <sub>v4.0.1</sub>

## Auxiliary
- Detect-It-Easy <sub>v3.11</sub>
- HxD <sub>v2.5.0.0</sub>
- Spy++ <sub>v18.00.11101</sub>
