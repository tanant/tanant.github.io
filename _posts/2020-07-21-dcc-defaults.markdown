---
layout: post
title:  "Logging Defaults in Various DCCs & Presentation"
---
there ended up being three parts: [logging levels][pt1], [filters+handlers][pt2], and [dcc defaults + presentation][pt3])|

So as you can probably guess, I'd *strongly* advocate for a simple setup - every one of your library modules does this

{% highlight python %}
import logging
logger = logging.getLogger(__name__)
{% endhighlight %}

and you move on. Really. That's all there should be to it.

If you are orchestrating an environment like, say, you're trying to assemble the nuke stack? Then you insert root handlers at the top level with the formatters and levels you want to collect. Since you're in control of the environment then you should also be able to decide where/when you want to turn things off or break up propagation etc since it's your environment.

## Splitting logs and controlling output

By now, it should be obvious that you can 100% control who gets what and when, we'll log to disk, Elastic/Kibana, the console, etc etc etc. If you're finding yourself having trouble because you're being told you're *over* logging then before you look at reducing the verbosity (tempting as it is) check that you aren't suffering from a presentation problem. Early optimisation is a Bad Idea. That logging you didn't think was necessary when everything was going fine? Yeah. Now you've turned it off and you've got a 20tb fluid cache that took you 18 hours to get to where it broke has no forensics because *you turned off the verbosity*. So, what can we do? We can split our logs. 

In nuke, for example, we have too much logging for people to know what's up, often there's a stacktrace in the console when something breaks but really, I do not expect artists to be watching a console. Some of our failure modes can be silent (ish) though, so maybe I can just make sure that the critical ones get bubbled up...

{% highlight python %}
class NukeAnnoyingHandler(logging.Handler):
    def emit(self, record):
        nuke.message("Uhm. Bad things might have happened. Talk to Anthony please\n\n" + self.format(record))

nah = NukeAnnoyingHandler()
nah.setLevel(logging.CRITICAL)
logging.getLogger().setLevel(logging.DEBUG)
nah.setFormatter(logging.getLogger().handlers[0].formatter)  # this me being lazy..
logging.getLogger().addHandler(nah)

logging.getLogger("chickens").info("eating grass")
logging.getLogger("chickens").warning("studying engineering")
logging.getLogger("chickens").warning("studying chemistry")
logging.getLogger("chickens").critical("have assembled a catapult")
{% endhighlight %}

Now, anytime you log anything that's CRITICAL it'll jump out in front of the user. Obviously, this is annyoing. But that's the point for a CRITICAL failure! You can extend this as well to hook in your logging with an auto-JIRA ticket submitter or some kind of help button that can capture process state and help people log better tickets or point them at a self-help troubleshooter guide. 
![a dialog box indicating that we have a critical popup](/assets/images/logging_artist_view.png)

In this case, had someone called me over, i would've spotted pretty quickly that the chickens had started studying engineering as the root cause leading to a catapult. I did not need them to anticipate that chickens studying may have lead to a catapult issue. 
![screenshot of a terminal showing more detailed logging messages](/assets/images/logging_developer_trace.png)

Not only that, since you can have multiple handlers out there quietly doing their thing - there's nothing stopping you from logging at crazy levels to local disk (performance notwithstanding, but that's a whole 'nother chestnut). When the artist sings out you can simply remote over, and check the logs thereabouts to see if there's a smoking gun. 

## A DCC Survey

For fun, I thought I'd poke around a couple of our core DCCs to see what vanilla behaviour was. Here's the scraping code I used:

{% highlight python %}
import logging
for k in sorted(logging.Logger.manager.loggerDict.keys()):
    if isinstance(logging.Logger.manager.loggerDict[k],logging.PlaceHolder):
        handlers = []
    else:
        handlers = logging.Logger.manager.loggerDict[k].handlers
    print "!!!!" if not logging.getLogger(k).propagate else "    ", k, handlers
print "(root)", logging.getLogger().handlers
{% endhighlight %}

#### Nuke 11
Hiero is pretty chatty as you'll find out fast, but there is a default logger, and it will dump to console and the script editor. Handy! Intrestingly the frameserver is not propagating. This is a Foundry default decision, and they have put a specific handler to cover that side of things.
```
     Hiero []
!!!! frameserver [<logging.StreamHandler object at 0x7f89757c00d0>]
     frameserver.FrameServer []
     frameserver.RenderQueue []
(root) [<logging.StreamHandler object at 0x7f8940376dd0>]
```

#### Maya 2019
Maya as well  (the other DCC I tend to muck around a little in) has the same setup. You can even see how they do split logs - one to the console, and the one that outputs to the gui. The StreamHandler is at level 50, and the MayaGuiLogHandler is at 0, so now you know why one only does CRITICAL and up messaging
```
     maya []
     maya.app []
     maya.app.renderSetup []
     maya.app.renderSetup.model []
     maya.app.renderSetup.model.observableProxy []
     maya.app.renderSetup.model.renderSetupPrivate []
     maya.app.renderSetup.model.sceneObservable []
     maya.app.renderSetup.model.selector []
(root) [<logging.StreamHandler object at 0x00000182036E7208>, <maya.utils.MayaGuiLogHandler object at 0x00000182023DD080>]
```

#### Houdini 18
No root logger :sadcat:. And huh, didn't realise all the other inbuilt stuff. Notice how there's no real hierarchy? That's likely why all those loggers are required at the points they are.
```
     bookish []
     bookish.flaskapp [<flask.logging.DebugHandler object at 0x7f2b37f85190>, <logging.StreamHandler object at 0x7f2b37edaf10>]
     bookish.pipeline []
     bookish.search [<logging.StreamHandler object at 0x7f2b37c0c710>]
     bookish.wikipages [<logging.StreamHandler object at 0x7f2b37cf4410>]
     future_stdlib [<logging.StreamHandler object at 0x7f2b398783d0>]
     webresponder [<logging.FileHandler object at 0x7f2b31228fd0>]
     werkzeug [<logging.StreamHandler object at 0x7f2b37edaf10>]
(root) []
```

#### Clarisse 3.6
I guess, at least it's clean.
```
(root) []
```

#### 3DE r4
This is kinda what I expect from 3DE, truth be told. I mean, 3DE is amazing, don't get me wrong.
```
(root) []
```

#### Blender 2.83.1
Aaand blender too.
```
(root) []
```

[pt1]: /2020/07/18/python-logging.html
[pt2]: /2020/07/19/python-logging-filterhandler.html
[pt3]: /2020/07/21/dcc-defaults.html