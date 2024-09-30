---
layout: post
title: "AppsScripting some logging for my tracking sheet"
---

Welp, that was fun - there's now a version that will, if you have the feature on, take a snapshot of the key values every time you update something. You'll need to be online is the main note however this was initially developed for a long indoor roll on Zwift/indieVelo so that shouldn't be an issue. The core tracking/time sheet thing will still work disconnected as per the core version too.

[![spreadsheet?](/assets/images/boring_spreadsheet_image.png "get it here if you want") ](https://docs.google.com/spreadsheets/d/1iWFNZhW3svgNRwQrqnh_LmJF14G-5v6g9XComEdOvhs/edit?usp=sharing)

[https://docs.google.com/spreadsheets/d/1iWFNZhW3svgNRwQrqnh_LmJF14G-5v6g9XComEdOvhs/edit?usp=sharing](https://docs.google.com/spreadsheets/d/1iWFNZhW3svgNRwQrqnh_LmJF14G-5v6g9XComEdOvhs/edit?usp=sharing)

It effectively superseeds [the original](/2024/09/25/spreadsheet-feed-tracker.html) if you're OK with AppsScript running on your account.

----

### AppsScript Architecture
So this is actually something slightly different that my original, local-only setup which was Excel based (excel is legit it's own programming environment, [like powerpoint](https://youtu.be/uNjxe8ShM-8?si=NLKu2TdX9Cg-9dGT)). That was a lot simpler as the model is strictly local execution and evertything packaged within the file and Excel is the host environment so _it_ deals with things. Theres no CPU quota to worry about either.

Not so for Google Sheets and AppScript. The arch is a very online-always kind of approach, even when the sheet is the host container. The execution is done *waves hands* over☁️there☁️somewhere🌈 which comes with it some awesome capabilties to blend things from other sheets and online resources.

It also means I have to deal with.. security.

### Permissions
For isolation puropses as a start the AppsScript[^1] is assocated with the sheet, so when someone copies it into their own world the whole thing should be consistent. That's great as I don't need/want centralised control over this thing, maintenance is just getting a new version. Everything here is transitory!

The original brief I had in my head was time-based logging as it's just a timer and I want datapoints regularly spaced out. I already have a minute timer to update, so that was the initial thought. I could use a time trigger like say..

{% highlight typescript %}
ScriptApp.newTrigger('capture_sheet_state')   // the function would check for a halt condition and instantiate again if needed
    .timeBased()  
    .after(5*60*1000)   // whatever frequency you want, but not insanely fast or else.. yeah
    .create();
{% endhighlight %}

This however requires a particular permission - `https://www.googleapis.com/auth/script.scriptapp`[^2] - and owing to the sheets/apps script architecture, it'll be running on a remote server. Like, I know I'm fine with the code (since I wrote it..) and it's simple enough to audit.. but also, that's a horrible expectation in this case. Like, look at this permission request. Not only do I need to auth my 'app', the script permission is broad enough to do pretty much anything.

![a marked up picture of a spreadsheet](/assets/images/permissions_request.png "ew, no")

Like, hell no.

Instead what I'm going with is a much simpler route - the `onEdit` simple trigger, which is a simple event that is a nice scope-limited thing to that sheet. I can use `https://www.googleapis.com/auth/spreadsheets.currentonly` as my oauth scope which is significantly less scary and more appropriate.


#### Silent failures
Another interesting note - if you were to use the `onEdit` trigger to invoke a function that requires elevated permissions compared to what has been authorised for the sheet then things will fail in a way that to the user, seems silent. I _beleive_ because we're running the `onEdit` event in Auth.LIMITED, there's only so much it'll do, and one of those things is to _not_ pop up a dialog asking for permission.

You'll get a dialog if you invoke a function via a button of sorts (that should go to the `appsscript.json` and check what scopes and ask for them all at one go), but if you don't silence. You'll only see the failure in your logs.

![error messages](/assets/images/ds_error_message.png)

So possible an interesting failure case - if you have something working with an install flow asking for permissions but down the line someone revokes them and you use simple triggers like this for hooking in, there's a strong chance you only will get failures in the logs.



[^1]: I realise it can't be App~~s~~ Script, but ugh.. AppsScript is such a pain to type constantly with the double 's'. 
[^2]: Unrelated peeve - https://developers.google.com/identity/protocols/oauth2/scopes doesn't list this scope. Why? 
