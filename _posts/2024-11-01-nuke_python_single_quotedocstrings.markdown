---
layout: post
title: Nuke and single vs double quoted strings
---

Recently I started working with [mattepaint.com](https://mattepaint.com) which has brought me (slightly) out of my usual pipeline hole, and back into the radiant sunlight of Nuke ~~bugs~~ quirks. You know how sometimes things just _feel_ sticky? Yeah.

So here's one - **don't use single quotes when doing python blockquotes.**  (as at posting time - verified in Nuke 13.2v7 and 15.3v1)

## Whut (symptoms)
Often we get used to ignoring errors in the console as they're not fatal or they just are expression links that are busted but you'll see this in the console as a *Syntax Error*:
![error message in the console](/assets/images/nuke_single_quote_console_error.png)
(In 15, you'll get that in the console and the script editor)

It's weirdly non-fatal at this point, your copy-pasted node is actually going to be fiiiiine so you won't immediately notice a problem if you've just introduced something like this.

## ..and now the issue
If you notice in the image above the dropdata is treated like a string.. a long string. And because Python is trying to be helpful, it's trying to underline the line with an issue with carets on the line underneath.

And if you're copy-pasting more than one thing, or maybe a thing with a bunch of curve data.. that string gets quite long...

And operations that write to the console have to finish before they return interactive control to you...

And yeaaaaaaahhh... 

<iframe class="video-container" src="https://www.youtube.com/embed/AsKhEcBLqf0?si=c5_roN_hwcJi2Bd9" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

All because there's a single node, somewhere in that copy-paste buffer that has a set of triple single quotes. 

## Fixing it!
Actually, it's pretty simple. For spot fixes, brute force check your scripts if you've got `'''` and replace with `"""`. Stopping the contamination at source is harder - treat it like you'd treat that ol RedGuard thing 😰. It's not quite _as_ bad as it's only within a localised area of the script.. but it's also harder to detect than RedGuard. Win some, lose some.

## Deep(er) dive.
No, really, when you're writing code on a python script button or you're doing large block comments you might want to use python's block quoting syntax, and in python you don't have a real distinction between " and ' so it's down to the type of muscle memory you might have developed[^1]
```
def function():
    """this is fine"""
```

```
def function():
    '''this is NOT fine'''
```

As noted by [Liam Collod](https://mrlixm.github.io/) who was kind enough to be my sanity checker the other day, it looks like in the copy-paste buffer Nuke isn't escaping the single quotes correctly. 

To play in nuke, here's a transform cut/paste with the problem:
```
set cut_paste_input [stack 0]
version 13.2 v7
push $cut_paste_input
Transform {
 name TransformBoo
 addUserKnob {20 User}
 addUserKnob {26 efo l "" +STARTLINE T "i have single quoted block comments"}
 addUserKnob {22 button01 l grr T "def something():\n    '''single normal block quotes'''\n    pass" +STARTLINE}
}
```

And here's the handcrafted fix
```
Transform {
 name TransformYay
 addUserKnob {20 User}
 addUserKnob {26 efo l "" +STARTLINE T "I still have single quoted block comments"}
 addUserKnob {22 button01 l grr T "def something():\n    \'\'\'single normal block quotes\'\'\'\n    pass" +STARTLINE}
}
```

👻🎃🧟👻🎃🧟👻🎃🧟👻🎃🧟👻🎃🕷️👻🎃🕷️👻🎃🕷️👻🎃🕷️👻🎃🕷️👻




[^1]: though, if you're getting into Python? Very Strong Reccomendation to use " as that's convention - see [PEP-257](https://peps.python.org/pep-0257/)

