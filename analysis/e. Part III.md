# Part III
In the previous part, we saw how methodology alone would break down when put up against a complex scenario with little to no context. What worked well for uncovering the earlier validation mechanisms was quickly diminished in the face of a relatively opaque routine that made even "brute forcing" unfeasible.

The last failure made me see clearly, that tools and techniques in Reverse Engineering were simply means to an end. The primary goal should be to piece together the original creator or developer's likely intention with the design of routines and mechanisms.

Every application can be thought to possess a "Story". An intended flow from start to end, from input to output. Especially for an app that didn't strictly require server-side validation, everything required for the proper and complete functionality of the app is found within its binary and the system it is hosted or installed on.

It should only be a matter of time and effort before every secret was unraveled.

With this new mindset, I set out to discover the "Story" of Puzzleball 3D in order to better my attempts at cracking the activation mechanism and if possible, fixing the typo.

It should be stated that Google Gemini was immensely helpful when performing research on historical application design philosophies and relaying that with information found with testing done on Puzzleball 3D in order to arrive at the final goal.

## The Activation Mechanism Story

