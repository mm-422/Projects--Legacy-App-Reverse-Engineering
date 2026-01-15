# PROLOGUE
> A summary of the approach and initial goal.

## My reasoning...
I set out to tackle a challenging branch of cybersecurity - reverse engineering - with a "real" application. Not just a simple "crackme" or "tutorial program". I wanted to build some confidence in this area of security which isn't widely taught. And while it did take some grit to weather the first couple of failures, I eventually achieved all the goals I had for this project.

## Some history...
Puzzleball 3D is a video game that was distributed on the Windows platform in the early 2000s. A free trial version was made available that used an activation mechanism based on CD-keys.

I distinctly remember this era of computing being rife with "cracks" and "keygens" which were methods used to circumvent this type of copy protection and enable piracy. Puzzleball 3D was no exception. An old keygen for this app (no longer functional) can still be found on the internet.

The publisher for Puzzleball 3D held rights over numerous other titles and implemented a custom "launcher" to unify all their products under one banner. This launcher was dependent on an external and proprietary DLL file that implemented extra validation routines to fight against code tampering.

Some time in 2009, the app publisher made an update to this DLL file that broke all available keygens. Not too long after, they went out of business, leaving games like Puzzleball 3D permanently locked behind that launcher.

## A goal based on assumptions...
Part of the reason why I chose Puzzleball 3D for this project was the thought that I'd be able to restore an "antique" piece of video game history back to full functionality while also gaining real, practical experience with using RE tools.

I set my eyes on decoding the activation mechanism first, thinking that it would be a simple "bit flip". It was not until much later that I realized the need to change my approach from the more conventional route to something that would be more appropriate for an application put together more than 20 years ago, with no documentation for it whatsoever available.

## Let's go!
The first two parts will detail my initial attempts at trying to "break open" the application to see how it worked. The final part will demonstrate what finally worked and how I eventually got to that part.

I do want to reiterate that the purpose of this project is not to promote piracy. No "cracks" or tools to circumvent general copy protection is made available. Everything was performed under an educational lens.
