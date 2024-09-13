---
layout: post
title: "Measurement Protocol - Core Setup"
---
[Overview][page0] >> *Core Setup* >> [Validate Loop][page2] >> [MP Fun!][page3]

We'll be setting up a _new_ project, with billing from the Google Cloud side, then bolting Firebase in, and three apps will be generated - web, android, and iOS (it's an ankle-tap as you'll see later)

#### 1. Turn it all on
Head to the google cloud console ([console.cloud.google.com](console.cloud.google.com)), and set yourself up a project. Mine is `Errant Blueberry`.

Once you're done there, head to the Firebase console ([console.firebase.google.com](console.firebase.google.com)) and convert that to a Firebase project.

When it asks, yes, yes you do want Google Analytics for this project. For this playaround purpose, I've created a new account.. and lastly configure GA.

For 95% of users the defaults are probably buuuut I would prefer less access so I'm only letting anon data for benchmarking to move across. 

#### 2. Generate 'apps' (data streams), config, and service keys
Righto. So in the Firebase console, generate three apps. For each app, get the Firebase App ID. For the `web` app also grab the config pack. We will use this for the loop test in a bit, and we want the Measurement ID as well anyway.

![a screenshot showing configuration information](/assets/images/data_to_grab.png "these things here")

Okay. Now we have to go to the _analytics_ console so visit [analytics.google.com](analytics.google.com) and crack open the Admin view.

From there, under Property Settings > Data collection and modification head to Data streams. For each of your Android, iOS, and Web streams, you will need to generate a Measurement Protocol API secret. Note that these are linked _to that particular stream_ so keep track of what belongs to what.

![arrows highlighting the admin icon and the data stream option](/assets/images/google_analytics_where_to_go.jpg "admin cog took me longer than I like to admit to find one time :|")

⚠️ One HUGE gotcha, there seems to be a width bug so don't do what I did and copy the text manually.. you could lose a few characters. Make sure you can see the copy icon.

![two keys, one cut off badly](/assets/images/width_bug.jpg "yes. I did fall into this trap")

Fiiiinally, in admin, look up your property details and get the property ID. You don't need this, but we're impatient and will use this in a second because who's got time to wait?

All up you should have something like this, for _Errant Blueberry_ (458451778):

| app      | firebase appID | gtag ID | MP API Secret[^2] |
| -------- | -------- | --------------- | ------------- | 
| web      | 1:517598319945:web:ed1a0479f89ed18962b698 | G-6Y3DPC68XD | pWumcEQ6QNiz1uwyf9Wn6w |
| iOS   | 1:517598319945:ios:1f478ed2c562d26c62b698 | - | M731bG3zTiOCEnyuC4s1Sg |
| android  | 1:517598319945:android:8a6fb54383a785c962b698 | - | Qdk40xCuRK-u5PQiWb4vqQ |

If it wasn't obvious, the AppID contains an identifier for the project (I think), the platform type and then some kind of UUID-like.
  
#### 3. Bigquery link
Almost done with config! Back to the firebase console and the project's settings, you want to manage Integration. Since we made our apps beforehand it should have 3 of 3 exporting (but make sure).

![screenshot of the initial bigquery link dialog](/assets/images/base_bigquery_setup.png "if you do the link first i think the apps don't default export. I forget.")

You'll also notice if you've not got billing, Streaming won't be an option. It's not critical but I would Strongly Suggest for the prototyping phase (for the scale I'm working at I'm still in the free tier). Waiting an entire day to do some checks is just unmitigated pain.

Finally, if you've _just_ set this up you will notice that the dataset is not created. Let's fix that.

![screenshot showing "dataset not created"](/assets/images/no_dataset_created.png "pfft. let's just create it.")

Originally I waited overnight for the dataset to be created as regular batch operation.. but you can just do it yourself now, if you know the dataset name it's looking for which is.. `analytics_{propertyID}`[^1]

So head to the bigquery warehouse for your project, and well, make that dataset. You'll know it's working correctly when you see a new tabled called `events_intraday_` appear, and back in the firebase console it'll show that dataset linked across.

![screenshot of bigquery setup](/assets/images/bigquery_lazy.png "tadah!")

>❗If you aren't set up for streaming you won't see this! You will need to wait aaages and there's no reason for this shortcut.

Phew! Okay, Time to move to Part 2, we've got all our config sorted. 



[Overview][page0] >> *Core Setup* >> [Validate Loop][page2] >> [MP Fun!][page3]

[other_article]: /2024/08/11/firebase_analytics.html
[page0]: /2024/08/12/measurement_proto.html
[page1]: /2024/08/12/measurement_proto-setup.html
[page2]: /2024/08/12/measurement_proto-loop.html
[page3]: /2024/08/12/measurement_proto-actually_posts.html






[^1]: Was quite chuffed to work that out for this tutorial. The hint was from reading [https://firebase.google.com/docs/projects/bigquery-export?product=analytics#change-dataset-location](https://firebase.google.com/docs/projects/bigquery-export?product=analytics#change-dataset-location)
[^2]: yes, these are all revoked 😂
