---
layout: post
title: "Measurement Protocol - Actually POST"
---
[Overview][page0] >> [Core Setup][page1] >> [Validate Loop][page2] >> *MP Fun!*

Aaaaaaaand now the actual thing you're after - since we're just using straight up POSTs you can use whatever your language of choice is and your engine of choice. Since some of my audience is likely still VFXies and we're heavily on Python i'll do that for now but you can totally see what's going on I bet.

![alt text](/assets/images/gtag_or_firebase.png "title")

If you prefer an interactive thing to play around with, I'll suggest having a look at the [GA4 event builder](https://ga-dev-tools.google/ga4/event-builder/) initally. With the previous two steps completed it should be as simple as switching between the correct message format (gtag vs firebase) and you can eyeball the requests it makes to understand what's going on.

In more detail.. here's what the core payload looks like. Events are an ordered list, each with a name, and parameters you pack. The `user_id` is an identifier you choose as you wish.
{% highlight python %}
core_payload = {
    "user_id":"Oscar the dog",
    "events": [
                {"name":"mp_ping", "params": {"occasion":"celebration", "cakes":5, "some_fractional":1.7}},
                {"name":"mp_pong", "params": {"another_arbitrary_thing":"strings are fine, bools are not"}},
                ] 
    }
{% endhighlight %}

For web apps, the framework leads to gtag land and you need to supply a client ID. This has been to date faaaaaairly freeform - I would probably copy the standard form but for this demonstrator purpose, you can make things up.
{% highlight python %}
gtag_payload = {"client_id":"abcd up to you really"} | core_payload
gtag_url = "https://www.google-analytics.com/mp/collect?measurement_id={measure_ID}&api_secret={api_secret}".format(measure_ID="G-6Y3DPC68XD", api_secret="pWumcEQ6QNiz1uwyf9Wn6w")
req= urllib.request.Request(
    method="POST", 
    url=gtag_url,
    headers={"Content-Type":"application/json"}, 
    data= bytes(json.dumps(gtag_payload), encoding="utf-8"),
    )
urllib.request.urlopen(req)
{% endhighlight %}

In Firebase the same concept is the an app instance (makes sense - it's more install-headspace). This needs to be a 32-len hex string (see below however). This ID should be pretty novel to analytics so this confirms that we don't need any kind of setup phase from a legit app anywhere and we can generate our own arbitrary idents.
{% highlight python %}
firebase_payload = {"app_instance_id":"abcdbeefdeadbeefdeadbeefdead0000"} | core_payload
firebase_url = "https://www.google-analytics.com/mp/collect?firebase_app_id={app_ID}&api_secret={api_secret}".format(app_ID="1:517598319945:ios:1f478ed2c562d26c62b698", api_secret="M731bG3zTiOCEnyuC4s1Sg")
req= urllib.request.Request(
    method="POST", 
    url=firebase_url,
    headers={"Content-Type":"application/json"}, 
    data= bytes(json.dumps(firebase_payload), encoding="utf-8"),
    )
urllib.request.urlopen(req)
{% endhighlight %}

If you shoot those two off, you should then see the events populate in your bigquery stream (you did set that up right? this is why you set that up.)

![alt text](/assets/images/query_results.png "title")

## Field mappings and curiosities
In normal operation `user_pseudo_id` is drawn from the gtag `client_id` or the `app_instance_id` hex string you supplied. While the `client_id` is fairly relaxed, if you supply a malformed (short/long) instance ID the event will disappear into the ether as far as I can tell.

However.. while writing and checking my code I got the two mixed up 🤦 and was wondering what the heck was going on with data not flowing through as expected.. turns out if you use the `client_id` key but an android or ios endpoint.. you can use an arbitrary string again? 

![alt text](/assets/images/client_id_usage_oddity.png "title")

I would probably not do this. This feels weird on so many levels. _So many_.

## DebugView/Debug mode
If you want, you _can_ pass in a flag for debug mode on these MP events to get information flowing into the DebugView.. but there's a few gotchas. You need to pass in the flag per-event, it's not a global call on the request (this matches how the payloads look in gtag calls I've inspected though) *and* you then need to poke the DebugView mode.

I'll pull my comment from [Simo Ahava's post here](https://www.simoahava.com/analytics/debugview-with-ga4-measurement-protocol/) where I first got the hint at a per-event debug flag. 

>... I think it's just around some backend plumbing with the data provider feeding the view. I can get MP events in the debug view once I trigger off an event through regular gtag.js call, and interestingly, I can fire off a number of MP events with an arbitrary client id, then fire my gtag debug call, and all the older events will show up, along with a new debug device so I'm guessing backend filtering issue.
>
> I also just now used an ancient client id that was working from the morning, and it also doesn't appear in the debug view (but i can pick things up in the bigquery stream) which further makes me think it's an implementation detail where the function to add to that filter for client IDs in the 30 min window for debug view is not hit by MP.

So yeah. It's a pain for those of us wanting to use MP in this way which, to be fair Google does say that this is to augment regular data collection and not to replace it 😅


[Overview][page0] >> [Core Setup][page1] >> [Validate Loop][page2] >> *MP Fun!*


[page0]: /2024/08/12/measurement_proto.html
[page1]: /2024/08/12/measurement_proto-setup.html
[page2]: /2024/08/12/measurement_proto-loop.html
[page3]: /2024/08/12/measurement_proto-actually_posts.html
