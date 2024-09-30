---
layout: post
title: "Google-sheeting up a directeur sportif"
---

As part of preparation for a marathon one of the boxes I wanted to tick off was mental capacity to do _a_ thing across a long duration. Hello, there's a Chasing Tour race/ride - CT Gold to be exact - of 273km. [Turns out it was about 8h33](https://www.strava.com/activities/12055074688). _Perfect!_

While slightly overkill for anyhting else, putting out what would normally be a not-that-stressful 160W on average gives someting on the order of about 5,000 calories. As my [marathon run showed](https://www.strava.com/activities/12477672773), actually eating/drinking sufficient calories becomes a thing if I'd like to stay performant for as long as possible[^1]. 

So the brief was to show:

- what my upper and lower bounds are for my various consumption targets
- _where_ I am with regards to calories/fluids/sodium to those numbers, with regards to time;

and 

- I don't want to really have to think too much (red bad, green good); or
- be overwhelmed with data or have any complex things to wrangle.


[https://docs.google.com/spreadsheets/d/1pxMLvSKvFpFlk3SM5MiF_iFtmISedmcFlyy5c0qGiGY/edit?usp=sharing](https://docs.google.com/spreadsheets/d/1pxMLvSKvFpFlk3SM5MiF_iFtmISedmcFlyy5c0qGiGY/edit?usp=sharing)
[![a marked up picture of a spreadsheet](/assets/images/auto-nutri-ds.jpg "get it here!")](https://docs.google.com/spreadsheets/d/1pxMLvSKvFpFlk3SM5MiF_iFtmISedmcFlyy5c0qGiGY/edit?usp=sharing)

You'll want to take a copy and update it for your own purposes, of course. A number of cells are protected so it'll warn you if you're editing something that probably shouldn't be edited and if something does look weird or you want a formula explanation sing out.

## Observations and notes
During the race I was suprised by just how robotically I had to eat in order to get in the approximate calorie targets I had, and just how quickly I got food fatigue. The other big thing was how easy it was to _over_ drink. Wasn't a medical issue or anything, but you did have to run to the bathroom if you did that. 

Also, while bashing out this sheet and polishing it up, I'm struck by just how much the last 20% polish takes in terms of making it work nicely. As the dev, I know how it all works so it was super janky on the day.. but to release it? Oof. I found a lot of little things (font choice, sizing, the things below) make it a lot nicer to work with. Lessons for software dev in general, does track with how much time I put into making UX for artists that I hope they never actually notice because it Just Works...

### Technically interesting details (and annoyances)

- **DateTime**: If you give google sheets a field containing a date + time, and someone clicks in the box, google nicely gives you a date picker. It also not-so-nicely wipes out the time. This is why H9 and I9 are separated
![screenshot showing a calendar picker](/assets/images/ds-calendar-picker.jpg)

- **Formatting rules for durations**.. aren't stricly durations, they're more day fractions - beyond 24h you need to strip out days yourself otherwise formatting will make it weird. I encountered this on my [CT Yellow](/2024/07/06/chasing_yellow_analytics) bigquery thing now I think about it..

- **Conditional Formatting**: Oy. To get this working, since I wanted a colour range - cell C3 (and D3, and E3) based on varying values, google doesn't allow ranges with formulae. So what I'm doing is using the percentage approach and including my min/max values. I then put two other single-colour rules in front so if those are hit (you're eating too little or too much) those take over.
![](/assets/images/conditional_formatting.jpg)

- **Auto-update**: There's a function in google sheets that lets it recalculate formulas every minute so that keeps the thing ticking along so you don't have to interact with it necessarily. ![spreadsheet settings](/assets/images/calc-every-minute.jpg) It's not critical as you can you can simply  bump a cell and it'll have the same effect - you'll have to do this on mobile.[^2]

### Upgrades ➡️ logging

⚠️ Edit: 30 Sept: [aaaand done!](/2024/09/30/spreadsheet-autologger.html) ⚠️

During the ride I realised I would also liked to have a little bit of historical tracking - I'm currently converting the sheet to be online in Google Docs, with some AppsScript bolted in that should record data every 5 mins or every time something gets updated.

It'd be interesting to get a graphical picture of my timings about when/where I ate what. Doubt there's anything earth shattering there however it'll be interesting to see overshoots and if I suddenly panic and eat a whole lot to catch up (or if I feel like meh and try cover it with cans of fizzy drink).


[^1]: This will also come in handy if I decide to do a vEveresting next year since I project _that_ at like 15-18 hours and not snacking properly will make me feel like absolute rubbish.
[^2]: It's also built specifically so that you can carry it as a document on your portrait-oriented mobile phone, you'll not get autoupdating but if you're out doing a long jog then this still works conceptually as a "wait.. how am I actually tracking vs my plan?" if you have some standardized units of eating you have in your bag.