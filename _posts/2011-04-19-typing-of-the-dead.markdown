---
title: Getting Typing of The Dead to work in Vista/Win7
redirect_from:
  - /notes/2011/04/19/getting-typing-dead-work-vistawin7
---


### (For the impatient)

Problem
: You can't type apostrophes and your right shift key doesn't work in Typing of the Dead.

Cause
: You're running Vista/Win7 which is treating your keyboard properly as a 106-key keyboard. Typing of the Dead, on the other hand, was built to expect a 101/102-key device.

Solution
: Override Vista and tell your keyboard hardware driver to mimic a 101-key device. 


1. Fire up regedit.
  - go to `\HKLM\System\CurrentControlSet\services\i8042prt\Parameters` (take note of what's there, you might want to reset this at some point!)
  
1. You want to tell Vista your keyboard is an oldskool 101Key unit.
      - set `OverrideKeyboardIdentifiter` to be `PCAT_101KEY`
      - set `OverrideKeyboardSubtype` to be `0x0000000`
  - note: steps may not be 1-for-1 on your machine, so YMMV. 

1. Reboot.

Caveats of course:
You're playing around in an area to do with setting keyboard overrides. It may, for all I know, make your keyboard unusable til you undo the changes, so make sure you're comfortable resetting this change without the use of the keyboard if that happens. That suggests you probably should be comfortable using the OSK to login, and have a copy of the registry file ready to import via 2-3 mouse clicks.

You should now be able to play Typing of the Dead. Go play for a bit, and then if you're curious, come back here and read on for more thoughts on what I think you've just done, my theories, and thoughts. 

#### For the curious

Welcome back - hope you enjoyed the pure unadulterated joy of type-based zombie slaying.

Anyhoo, onto the theory. I presume somewhere along the line in the great XP - Vista (and Windows 7) migration, Microsoft tidied things up and did a raft of improvements to the various subsystems in windows - one thing in particular being keyboard support. I have a feeling that up to XP support for weird-o keyboards and ones with extended keysets was done outside of core and required manufacturer drivers for proper support. I'm guessing that post XP, a lot of this support was moved into core to make people's lives easier, I mean, how many people really expect to install a keyboard driver these days?[^1]

What this meant for older software is a bit patchy. For TOD, it resolved itself as the apostrophe/quote (Unicode `0x0027`) character not firing and the the right shift key not seeming to to work (i.e. RtShift-1 doesn't type out an '!', just a 1). This isn't a problem til you start getting further in and racking up that perfect bar, but once you get one, there's nothing you can do but get hit. Bah.

#### Dead ends

My first guess was a simple keymapping issue - perhaps Vista/Win7 was firing smart quotes instead of the bog standard apostrophe. I therefore got hold of Microsoft Keyboard Layout Creator and did some poking which revealed that it was accurately sending out the correct unicode character. I did some testing which involved mapping the apostrophe key to the letter 'e' (which worked) and mapping the 'e' key to Unicode `0x0027` (which failed), and trying out the other smart quote characters (which also failed). Bit disappointing, but it also clued me in on something - the right shift key wasn't mappable. This suggests there might be something deeper at work.

#### Inspiration

I kinda shelved the project for a few days - had some other work to do - but came back onto it one night when I felt like some mindless game time. It was bugging me that I was close, but not quite there.. I hadn't yet made the connection to wig with the keyboard driver, but somehow[^2] I ran across [KB927824 - _Windows Vista may not use the correct keyboard layout when you connect a USB keyboard to the computer_][kb927824] (note: MS appears to have deleted this article, so this is a link to PKI Solutions' mirror of it)

Its got instructions on how to solve a problem with say, a JP 106-key device being ID'd as an English 101/102 key device. For a JP keyboard user this is a right royal pain. For me, it told me exactly what I'd need to do to force Vista's keyboard driver to do something slightly non-standard.

As soon as I saw that bit, took about 5 mins for me to save my other work, do a quick assessment, and then make the necessary registry edits. Success!

#### Yes, Dvorak _does_ work

Additionally, there seems to be some notes out on the interweb from people saying that TOD can't recognize their dvorak layouts. Er, that's wrong. It respects weird keymappings, and it's not hardcoded.

What's probably happening is that when you fire up TOD it's looking for and picking up your default keyboard config - new application instances will use the _default_ keyboard layout - which is probably QWERTY if you're learning Dvorak... what you will have to do is install the relevant keyboard and assign a hotkey combination to shift between input keyboards so that when you fire up TOD you can switch to your desired layout. 

#### So, what's actually happening under the hood?

In short, dunno.

I figure TOD is still leaning on the host OS for keycodes (as evidenced by me being able to remap out to dvorak and have it work) as opposed to just taking raw keyboard events, but then there's the open question of why the keyboard driver then makes a difference. There's an outside possibility that TOD is intercepting keyboard events and discarding the ones out of a particular safe (or unrecognized) range. This would explain the behaviour as there would be what, 5 keys that TOD would be discarding (they look to be: right shift, apostrophe, left square, equals sign, tilde)?

The fact there are some other keycodes that aren't percolating ~~(# for example)~~ (# works, I had accidentally set the keyboard to UK instead of US) still worries me, and suggests I don't have it completely sorted, will have to do some more prodding, but the fact I have to reboot slows me down a tad. Anyone knows more about the windows keyboard system, can they  [let me know][mailtome]? I'm curious...


[^1]: If you're ever playing around with keyboard layouts the Microsoft Keyboard Layout Creator (v1.4 as of this writing) is rather handy. You can just do up your own keyboard with special keys for things like em-dashes, weird type symbols, and stuff that you'd normally have to memorize the Alt code for, or use the character map.

[^2]: Coincidentally, the other day I was reading an Alertbox column on poor search skills making problem solving difficult. This problem is an example of where no amount of googling would ever show the answer (well, now it might that I'm posting this...) so alternative search techniques are required. KB927824 came up as a result of me using a search engine to find some other possible hits, and reading in depth to learn more about the topic. It's an example of I guess, not being completely slaved to the search engine as answer engine paradigm. Now I know what registry keys I'm looking at, I can start hunting through Technet to find out exactly what I'm changing and doing..


[kb927824]: https://mskb.pkisolutions.com/kb/927824

[mailtome]: mailto:{{ site.email }}
