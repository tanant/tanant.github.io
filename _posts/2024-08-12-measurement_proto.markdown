---
layout: post
title: "SDKless Firebase/Google Analytics Measurement Protocol "
---
*Overview* >> [Core Setup][page1] >> [Validate Loop][page2] >> [MP Fun!][page3]


I'm probably coming at this from a very non-standard angle, as my _first_ data source is not an Android, nor iOS client, and it's not a connectec web app.. It's Autodesk Maya. This is the detail of the setup I mentioned [here][other_article] earlier. 

More specifically, I would like _an_ analytics capability that lets me capture information for an artist's session - what tools are being used, what's exploding, what's being touched, context switches, key events, that kind of thing. I also want some logstash-like behaviour for better remote troubleshooting but that's down the line. I would also like to ~~blame~~ shout out [Mike Hardy](https://github.com/mikehardy) for suggesting this when I mentioned my use case.

Now I've gone through all the tech and worked it out, this actually feels somewhat simple in all honesty but I recall the repeated ankle-taps I faced due to flaws in mental framework, or issues with not quite getting the wording of documentation or poor assumptions so I wanted to write this all up as an end-to-end process.

## Steps

#### [1. Core setup][page1]
- Start services + accounts + do administrivia
- Generate endpoints/streams, keys. Collect all the configuration details
- Link to bigquery for later data extraction

When this is done, you should have all the configuration you need, along with a small stack of secrets written down and are ready to validate your loop. 

NOTE: You don't _need_ billing enabled, but it'll make your life a _lot_ easier in subsequent steps if you can stream data to bigquery.

#### [2. Simple event generation loop to validate plumbing][page2]

- Fire off a gtag event
- Pick up the event in the debug viewer
- Pick up the event in bigquery

Data flows from your fake client frontend and is picked up at the back to prove we have a working core loop. This is _not_ MP, this is regular GA stuff.

#### [3. Measurement Protocol fun!][page3]

- Field mappings, request format
- Debug view misery (suggest: avoid)

It's done!

### What exactly?
So in specific detail, what this series of documents covers is the setup, validation, and use of Firebase/Google Analytics _Measurement Protocol_ and doing **everything** production-ready outside of the APIs.

By the end of this process you should be able to fire off POST hits with arbitrary data, arbitrary identifiers, and pick all the hits up at the other side for your own adhoc processing desires.

Later down the line, if you want to bolt in more well-formed setups (i.e. properly sessioned analytics via gtag) or leverage some of the madness of google's analytics? You can pretty much grow into it as desired.

### Why would you do this?
Maybe you don't have a framework that the SDK supports? (python..)

Maybe you have a case where you need to be incredibly respectful of what data gets collected and so can't use the SDK?

Feel like noodling around?

*Overview* >> [Core Setup][page1] >> [Validate Loop][page2] >> [MP Fun!][page3]


[other_article]: /2024/08/11/firebase_analytics.html
[page0]: /2024/08/12/measurement_proto.html
[page1]: /2024/08/12/measurement_proto-setup.html
[page2]: /2024/08/12/measurement_proto-loop.html
[page3]: /2024/08/12/measurement_proto-actually_posts.html

