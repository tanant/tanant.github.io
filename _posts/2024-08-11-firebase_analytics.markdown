---
layout: post
title: "Firebase/Google Analytics: enabling debug mode sans chrome extension"
---

TL;DR: Using the JS web API? You're effectively in `gtag` land, so use the `debug_mode:true` flag
```
    const app = initializeApp(firebaseConfig);
    const analytics = initializeAnalytics(app,{'config':{ 'debug_mode':true }}); 
```
And then you should be able to see events flow into your DebugView. 

---

I've been recently playing around a little with GA4 analytics as I would like to leverage some of the Measurement Protocol stuff to instrument something[^1] and of course while you're developing and playing analytics one of the first things you want is to set up is some kind of stream so you can see if you really are capturing what you want.

If I was developing something standard like an iOS or Android app, then yeah, your 'context' is an application and you pass the correct flags in, but I really didn't like seeing this:
![picture of the docs with the link to install an extension underlined](/assets/images/firebase_debug_mode.png "seriously, what?")

I have two grumbles here:
1. It's a chome extension.. what if I'm not running a chome derivative? (also, as of this post - the extension hasn't been updated in a while and I'm not confident yet that it's actively maintained 😑)
2. Why is it an _extension_? Aren't we just using the `gtag.js` system in effect? (spoiler: we are)

From the [API docs](https://firebase.google.com/docs/reference/js/analytics.md#initializeanalytics_a68c1d7), the function signature is `initializeAnalytics(app, options)` where `options` are `AnalyticSettings` and that interface takes in a `config` property, which looks like this:

![picture of the docs with the link to install an extension underlined](/assets/images/firebase_documentation_enumeration.png "I'm not sure why these specifics are enumerated")

Because the text seemed so "this is the canonical set" I thought i'd need to pass that `debug_mode:true` parameter in another fashion. It wasn't working and so I had a look through the JS API on github only to notice that the interface wasn't exclusive and you could just shove whatever the heck you wanted in and then.. 💡

### Should I use the extension?
So there are still possibly good reasons to use the debugging extension if you want - the calls are _very_ different if you inspect the POST and payload (hint: there's no payload, it's all just a giant query string with a `_dbg` flag.. which you might actually find handy to know), and it also doesn't batch the queries - it'll send them one by one .


[^1]: Some tooling for Maya I'm writing to streamline an animated short - I'd like to anticipate end user issues _before_ they happen and get overall usage stats to tune where to put time/energy into.