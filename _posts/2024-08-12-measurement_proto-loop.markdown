---
layout: post
title: "Measurement Protocol - Validation Loop"
---
[Overview][page0] >> [Core Setup][page1] >> *Validate Loop* >> [MP Fun!][page3]

This should be relatively short, we're going to be using a gtag-based setup to get something flowing across. Literally this:

(yes yes, the project is deleted, keys are revoked, billing is disabled, etc. Was an actual decision to not redact things just because it makes documentation a bit easier to follow)

{% highlight html %}
<html>
    <head>
        <title>web-property</title>
        <script type="module">
            // Import the functions you need from the SDKs you need
            import { initializeApp } from "https://www.gstatic.com/firebasejs/10.13.1/firebase-app.js";
            import { getAnalytics, logEvent, initializeAnalytics, getGoogleAnalyticsClientId } from "https://www.gstatic.com/firebasejs/10.13.1/firebase-analytics.js";

            const firebaseConfig = {
                authDomain: "errant-blueberry.firebaseapp.com",
                projectId: "errant-blueberry",
                storageBucket: "errant-blueberry.appspot.com",
                messagingSenderId: "517598319945",
                apiKey: "AIzaSyCLjrjD4RFdFb241BLAnrpbs69RsTjPjZE",
                appId: "1:517598319945:web:ed1a0479f89ed18962b698",
                // measurementId: "G-6Y3DPC68XD" // yep. this is commented out!
            };

            const app = initializeApp(firebaseConfig);
            const analytics = initializeAnalytics(app, {config: {'debug_mode':true}}); 
            document.write(await getGoogleAnalyticsClientId(analytics));
            logEvent(analytics, 'chip_flavour', { name: 'bbq'});  
            logEvent(analytics, 'other_data', { fish: 'chips', potato:'raw', note:'yes', qty:12});  
        </script>
    </head>
    <body>
    </body>
</html>
{% endhighlight %}

## Things to note
1. You _will_ need to have a local webserver to be hitting, you can't access this as a straight up file for various reasons and will not send any analytics out to google. I just spun up a nginx container and bind-mounted to the directory.
2. In your browser, turn _off_ adblocking because that will probably block those calls.. 
3. Set either google analytics or your dev machine to be whitelisted in any DNS blackholing you're doing 😅

## Validation and what you should see
Things to check for - locally, if you're using the above html, it will write into the body of the document what client ID is associated with that browser. 

If you throw up a devtools sidebar and watch the network traffic you should also see (slightly lagged) a hit to `collect`

In google analytics or the firebase console, you should see something appear in the realtime view with your arbitrary events (and page_view too). In the debugview you'll also see the same things, and if you go to the bigquery store in the `_intraday` table, you should find your events.

![screenshots showing where events should appear](/assets/images/data_flow_validation.png)
## Other things of note from poking around
In our firebaseConfig, i've commented out the MeasurementID as it's not important. In the console, you'll notice that there's a request to firebase.googleapis.com with the appId that returns what is in effect, the config package with the measrurement ID.

If you change the appID to say, the ios version? The return from firebase lacks a measurement ID which renders the `gtag` side non-functional, this sorta kinda makes sense - firebase analytics for everything apart from web app is a thing, while web apps are piggybacking off gtag... which also means if you're using that stream you should just learn all about gtag.


[Overview][page0] >> [Core Setup][page1] >> *Validate Loop* >> [MP Fun!][page3]

[TOC]: [/2024/08/12/measurement_proto.html]
[page0]: /2024/08/12/measurement_proto.html
[page1]: /2024/08/12/measurement_proto-setup.html
[page2]: /2024/08/12/measurement_proto-loop.html
[page3]: /2024/08/12/measurement_proto-actually_posts.html