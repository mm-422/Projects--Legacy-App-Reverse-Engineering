# Project Summary
```
Tools and techniques are only means to an end,
  Understanding the intent behind an application's design is the ultimate goal.
```
Multiple times throughout the course of this project, the importance of understanding how an application behaves through gathering historical context and correlating that information with data gained from careful testing, was clearly demonstrated.

Often times, RE projects start with a quick search for an "entry point", followed by simple hacks to flip a result or outcome, and then ending with a fully unlocked or unrestricted version of an application.

With Puzzleball 3D, a decades old program with legacy design, it was when I tried getting into the mind of the original developer, that the tests and assumptions stopped leading to dead ends and began improving drastically.

Instead of following a typical and linear process, I began thinking about how the original creators might have designed the application and its components and what possible considerations/trade offs they may have had to make.

In the case of the activation mechanism, it was when concepts like "The Judge" and "Librarian" were formed to give structure to the often-abstract assembly, that the routines began to seem a lot less overwhelming. While the solution in the end may have been a simple "bit flip", the path towards that final deciding instruction was anything but straight forward.

``Arcade.dat`` also presented its own set of curveballs. One of them being the integrity-check mechanisms lined with duplicated strings in both the main executable and the primary DLL file, ``ra.dll``. A simple hex edit in the end was also thwarted by a proprietary EOCD sequence, which was discovered with the help of a behavioral analysis tool, Procmon.

While today's tools are very advanced and do a lot of the heavy lifting, the mind of the original creator is the most valuable asset one could possess when reverse engineering an application.

This is of course not quite possible to obtain. So the next best thing is to reconstruct what thoughts and considerations may have gone in to the building of an application by careful use of tools and techniques to compile the crucial historical and behavioral data.
