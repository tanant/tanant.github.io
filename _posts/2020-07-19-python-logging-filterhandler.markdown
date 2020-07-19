---
layout: post
title:  "Logging Handlers and Filters (for you VFX/FEAT people out there)"
---

Righto. So assuming you got the [basic logging][pt1] this is where I start getting into filters/handlers. I'm taking Maya as my baseline, just because I have it open and it's more likely you're used to Maya than say, Nuke. The same principles apply.

(Really, this could be just in the other piece, but there's often too much text in one go, so you know, easier to read this way)

### Setup
We're going to need to set up our environment thusly:

{% highlight python %}
our_handler = logging.StreamHandler()
chicken_fmt = logging.Formatter('POULTRY: %(message)s')
our_handler.setFormatter(chicken_fmt)
logging.getLogger("foo.magic").addHandler(our_handler)
logging.getLogger().setLevel(logging.DEBUG)
{% endhighlight %}

So this setup here is a little odd, in general, you're going to likely want your handlers all at the root level, but I'm putting this in here to illustrate that Handlers don't really participate in the hierarchy as such.

### Handler rejection of events

{% highlight python %}
our_handler.setLevel(logging.WARN)
logging.getLogger("foo.magic.chickens").info("hiding eggs")  # this is normal
logging.getLogger("foo.magic.chickens").warn("escape attempt about to start")  # this is not 
logging.getLogger("foo.magic.chickens").critical("escape attempt in progress")
{% endhighlight %}

Events of a level below that of the handler's settings are ignored by the handler. Note that we have still got the root logger still outputting things? That's because the handlers are not part of the logging propagation chain. They hang off to the side, handlers have no impact on whether or not events go anywhere, they are your 'output taps'. 


### Filters!

{% highlight python %}
# note, as of Python 3.2+ you can just supply a callable or a class with 
# a filter method. Handy!
class ChickenFilter(logging.Filter):
    def __init__(self):
        super(ChickenFilter, self).__init__()
        self.count = 0
    
    def filter(self, record):
        self.count+=1
        record.msg="bwarrrrk"
        return False

cf = ChickenFilter()
our_handler.addFilter(cf)
{% endhighlight %}

So. Filters. They can be used two do two distinct things:
 
 - tell the Handler to not bother outputting or handling a message
 - transform the LogRecord (!)

And if you want, you can use them to capture state or do other things as any other class can. Running our earlier three messages you should see these results:

```
# foo.magic.chickens : hiding eggs # 
# Warning: foo.magic.chickens : bwarrrrk # 
# Error: foo.magic.chickens : bwarrrrk # 

print cf.count
# 2
```

Notice that only two of the messages have been corrupted by the chickens - those at Warning and higher. The message below this level is let through as normal.. but all three messages are let through. This demonstrates how the processing stack is:
 
 - a handler checks if it should even bother.
 - if it should, then it can then modify the LogRecord and do something with it. 
 - even if you don't handle the LogRecord, you can certainly edit it!
 
The LogRecord always continues to the next Logger - if you want to filter out things, you'll need to apply a filter into the logging chain:

{% highlight python %}
class ChickenEscapeFilter(logging.Filter):
    def __init__(self):
        super(ChickenEscapeFilter, self).__init__()
        self.count = 0
    
    def filter(self, record):
        self.count+=1
        if 'escape' in record.msg:
            return False
        return True

cef = ChickenEscapeFilter()
our_handler.setLevel(logging.INFO)
logging.getLogger("foo.magic.chickens").addFilter(cef)
{% endhighlight %}

Note the minor gotcha here - you need to apply the filter at the logger level where the message was generated. If you were to apply the filter at `foo` or `foo.magic` you would see no impact.

[pt1]: /2020/07/18/python-logging.html
