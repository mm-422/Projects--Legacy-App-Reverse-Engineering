# PROLOGUE
> A summary of the reasoning and approach.

## Motivation
I wanted to explore reverse engineering - an often challenging area of cybersecurity - through a real-world application rather than a purpose-built "crackme" or tutorial-style test program.

Initially, the goal was to deconstruct a target application in order to understand its components ie. activation mechanism, validation routines etc.

While the early attempts were met with failure, I was able to refine my ability to reason about the intended design behind the target application and strengthen my proficiency with the relevant tools and methodology with each iteration.

## Historical Context
Puzzleball 3D is a Windows-based video game released in the early 2000s, with a free trial that relied on CD-keys for activation; a typical mechanism for the time.

This period of software distribution was also rife with "cracks" and "keygens", which were tools designed to circumvent and/or bypass legacy copy protection and enable piracy.

The publisher for Puzzleball 3D - who also held rights over numerous other titles - implemented a custom launcher tied to a proprietary DLL file in order to unify all their offerings under a single banner. The DLL file also contained validation and integrity checks in order to resist tampering. This became the target for "crackers".

However, in 2009, an update to the DLL rendered existing keygens and bypass methods non-functional. Not long after, the publisher also ceased operations, leaving video games like Puzzleball 3D locked in a permanent "free trial state".

## Initial Assumptions
One reason I chose Puzzleball 3D for this project was the assumption that its "archaic" activation mechanism would likely be relatively simple to analyze and even overcome with a "bit flip" or conditional check modification.

I initially approached the problem with techniques and methodology that would've been more suited to modern applications, which while a lot more secure, are also standardized.

It was only after repeated failure that I realized the need to reassess my approach. Aside documentation and symbols being absent, the application reflected design patterns and considerations that were specific to its era.

For example, simple assets for the launcher like text preceding an input field were copied multiple times into memory instead of being "pulled straight from the disk". This may seem inefficient by today's standards but to preserve good user experience, the original developers likely made this tradeoff as hard drives back then were relatively slow. This complicated analyis methods like tracing user input and decoding integrity checks, as will be demonstrated in a later part.

## Structure of This Analysis
The following sections document the entire process in stages:
- First two phases outline the intial approaches and why they failed.
- Final phase details the path that ultimately succeeded.

This project is presented strictly for educational purposes. No cracks, activation keys, or hack tools intended on enabling piracy are provided.
