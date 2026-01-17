# Phase I
> GOAL: Decode the activation mechanism.

## Reconaissance
- Installed Puzzleball 3D at default location under Program Files (x86) directory.
- Launching application from either .EXE or quick start icon presents launcher menu.
- [LAUNCHER IMG]
- Launcher contains various options and lists revision number; Rev. 3744 Build 177.
- [LAUNCHER BUY NOW TEXT]
- Text on launcher window indicates that game install is complete and is likely locked digitally.
- Clicking on "Other Games" button leads to an insecure and possible expired URL.
- [INSECURE SITE]
- Since site is no longer owned and not functional, SSRF and IDOR exploits are not possible.
- Tampering with the URL wont help with reaching the goal.
- CLicking on "Already Paid" brings a new window into view with 4 separate tabs.
- [ALREADY PAID MENU]
- Text indicates that internet connection is required to activate game.
- Intercepting application "call back to home" could allow for spoofing code validation.
- Clicking on "I'm not connected to the internet" renders new screen displaying a URL.
- [NOT CONNECTED]
- Product code is displayed here. This does not seem to change. Possibly tied to some property.
- Examining URL, product code is exposed in the parameter, "pid".
- May be possible to reverse-engineer and/or generate our own unlock code.
- Pay By Paypal option provides button for purchasing code through PayPal.
- Code will be emailed.
- Exploiting this channel isn't feasible as it requires server response with generated code.
- Pay By Web is similar to previous option but allows use of credit cards for payment.
- Quick Pay is similar to previous methods but allows for credit card payment through launcher itself.
- This skips the need for a browser.

## Static Analysis
