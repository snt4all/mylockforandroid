Step 1: Do a temporary keyguard disable when your process launches. In our case I do it at the start of the service where I'm assuming we are starting at boot, and again anytime the screen on broadcast comes in.

Step 2: At this point android waits for user to do things on your process, and then your activity to be done, at which point you must re-enable the keyguard, or else you will basically get blacklisted from further disables. The alternative to this is an action that "leaves" the present activity to do more/other stuff.

Step 3: So the API essentially provides this out as a function called Securely Exiting the keyguard. It's for activities that needed to happen while device was asleep (alarm clock is a perfect example of this) but then need to get out of that screen and do more. You can only call it after having done disable keyguard (which don't forget is basically only designed as a temp disable). What it is doing is checking in with the android lock gatekeeper, and saying to it: "We need to unlock. Prompt user for password if it exists." and when it doesnt exist it just lets you through (At the same time enabling the home key which is totally locked down and protected by the android OS at all times, and kept that way even when you are doing the regular disable keyguard command and no password exists.)

Step 4: When you actually do the secure exit, you get a callback when that succeeds. that exists for the case that user might fail to get the pattern right, your app sits and waits for it to be done. Then gets called back. So at the callback, you call re-enable keyguard which makes it airtight.

If you look at the source for welcomeactivity you will see the alternative method which depends on the activity code itself. that does exactly the same thing as the steps I just described, but it's a whole lot easier to use/understand, and you never have to use the re-enable command.

To get those steps to actually work I had to program a short delay between the temp disable and the real secure exit because the OS expects the commands to be separate. they never really thought to make an option for devs who did want to get out of the lockscreen for the purpose of calling up a substitute for it, just for those who might want to push some kind of dialog to the user in response to an event, with the eventual possible outcome of pressing a button to get out of that and into some other related additional task. It is very complicated and i now understand why it has to be given the way the pattern security is coded and the rules the lockscreen always has to follow with the home button. but to me it still seems uncharacteristically obtuse for it to NOT let you all the way out of the lockscreen just with the regular call when the pattern lock isn't even on. i'd think that would be the desired default non-pattern mode functionality.