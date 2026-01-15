# Prologue

## Reasoning
I initially set out on this project in order to tackle what I thought at the time was one of the hardest branches of cybersecurity. Programming and application "hacking" was something I didn't think I could delve into but fortunately, after a couple of failures, I managed to achieve all the goals when I first set out on tackling this project.

I didn't want to just reverse engineer a simple "crackme" or a "tutorial app". I wanted to tackle something real in order to build confidence in this area of cybersecurity. I was reminded of some abandonware applications that I recently came across and picked one of them to begin experimenting.

Puzzleball 3D is a video game for the Windows platform from the early 2000s. I remember this era being rife with "cracks" and "keygens" for unlocking full versions of applications. Puzzleball 3D was no exception. However, some time around 2009, the publisher had modified their launcher implementation to fight keygens and fortify their code from tampering.

A new keygen or crack was not made for Puzzleball 3D as it wasn't really worth anyone's time or effort. And so it was forever locked into trial mode unless you could somehow locate an older version. This is partly what lured me to pick this app for the project. I could learn reverse engineering and bring something back from a lost era back to full functionality.

I do want to reiterate that no "cracks" or binaries were distributed throughout this project. The goal is and was always to teach myself reverse engineering while also restoring functionality to an old "antique" video game.

Puzzleball 3D implemented an activation mechanism that was similar to systems utilizing CD-keys to enable full functionality. But as we'll see later, there is more than meets the eye. While I set my sights on decoding the activation mechanism, thinking it would lead to an easy "bit flip", the application would reveal later on how the devs went about implementing a custom, "secondary" validation routine contained in the proprietary DLL that acted much like a security guard for the main application.

I will detail my journey and the steps/methodology performed in three parts. The first two will demonstrate the pitfalls and deadends. The last part will showcase the importance of understanding app behavior before leading to success.
