---
layout: post
title:  "Python logging levels (for you VFX/FEAT people out there)"
---
there ended up being three parts: [logging levels][pt1], [filters+handlers][pt2], and [dcc defaults][pt3])|

So one of those topics that's been done to death a little but sems to cause some confusion is around logging in python. While it's a smidge confusing in the way the docs are written, it's quite a robust, well-featured system that should largely cover anything you need. There may be some situations where it doesn't - but that's rarer than you'd think and there's very few reasons for rolling your own. Honestly.

Personally, I'd really suggest going through the [official advanced logging docs][python-advanced-logging] but in case you have, and something isn't quite gelling, there are plenty of other excellent tutorials/discussions etc on python logging that are worth reading. My notes below are notes to myself, but might help if you're looking for a different phrasing or I express a concept in a way that works better for you.

There are two things you probably care about:

  - While developing & putting your library together, making sure you emit messages that will be useful to Future You. Likely, as well, you're going to also be drowning in log messages so you want to control what is and isn't seen. 

  - Finally, as a diagnostics tool, you're going to want to be able to get the log messages that help you solve the problem your artist calls you about in a blind panic at 5:45 on a beer-o-clock friday _without_ having to reun the job and sit there watching scrolling logs.

The former is about message control and filtering. The latter gets us nicely into handlers and presentation.

## Message control at the logger level

Loggers do the grunt work of moving LogRecords up the tree and are mostly what you'll be interacting with. These have two control points - `propagate` and their `level` - and are assembled in an internal hierarchy that's denoted by '.'s. It's like DNS.

Key things:
  - the level of the logger you're logging to is what determines if the message becomes a LogRecord and enters the logging chain;
  - the level can be _derived from_ the parent; and
  - loggers that arent the *specific* logger can choose to shuffle messages up or not. That is all. No filtering.
    

### The logger and it's own level controls event emission

This is somewhat intuitive, however it's worth just nailing down and making sure it's clear to illustrate the `logger` is just an entry point.

{% highlight python %}
logging.getLogger("foo.magic").setLevel(logging.CRITICAL)
logging.getLogger("foo.magic.chicken").setLevel(logging.INFO)
logging.getLogger("foo.magic.chicken").info("egg laid")    # Egg! Breakfast is saved!
{% endhighlight %}

As a starting point for LogRecords, there's no reason why the level of the `foo.magic` logger should have any bearing on the message flow. Why do I make this rather trivial example?

#### Logging level NOTSET means derivation from the parent

{% highlight python %}
logging.getLogger("foo.magic.chicken").setLevel(logging.NOTSET)
logging.getLogger("foo.magic").setLevel(logging.CRITICAL)
logging.getLogger("foo").setLevel(logging.INFO)
logging.getLogger("foo.magic.chicken").info("egg laid")    # wonder if we have any more eggs...
{% endhighlight %}

Setting the level at the parent in this case will have this effect, but to be really clear it's because the child logger searches up the tree for the level it should log at, not that setting data at the parent level pushes and sets loggers down the tree. Loggers seek upwards for this data.

{% highlight python %}
def where_is_my_level_set(logger_name):
    """Returns a text string of the logger that specified the log level.
    
    Returns:
        str: The logger name (a blank string if it's the root logger)
    """

    # technically, we could do an early bail here by testing the logger provied
    # but it makes the below code have an early exit path which isn't useful
    # in this sample code.
    parent_levels = logger_name.split('.')  # spit into the levels
    parents = []
    for x in range(0, len(parent_levels)):  # assemble into their names
        parents.append('.'.join(parent_levels[:x]))
    parents.append(logger_name)  # add the starting point
    parents[:] = parents[::-1]  # reverse in place, for fun
 
    # now check backwards up the tree
    for parent in parents:
        if logging.getLogger(parent).level != logging.NOTSET:
            return parent
    return ""
{% endhighlight %}

### Propagation is boolean

{% highlight python %}
logging.getLogger("foo").setLevel(logging.INFO)   # so everything down the line should be at INFO level
logging.getLogger("foo.magic").propagate = False
logging.getLogger("foo.magic.chicken").info("egg laid")    # Really wish I had more eggs.
logging.getLogger("foo.magic.chicken").setLevel(logging.DEBUG)  # Keeping on top of the chooks
logging.getLogger("foo.magic.chicken").info("egg laid")    # That cake isn't happening without eggs..
{% endhighlight %}

The message emitted at `foo.magic.chicken` level is not seen - this isn't due to a level issue, it's because we have stopped propagation. It's *a* way to manage this, but so far I've come across very few circumstances where this is a good option. If you're seeing spam at your root logger then turning off propagation for a chunk of the tree that is too chatty is a nice quick fix, but it's difficult for the next developer to know this (if they aren't seeing messages from some.stupidly.deep.module.path is it because you set propagation off somewhere? or is it because the module is quiet? or?)

Yes. You *can* set `disabled` on a logger, but it's not really part of the public API if you check python [issue#36318][disable-not-public] . Pls don't (just set the level to CRITICAL or something).


[python-advanced-logging]: https://docs.python.org/3/howto/logging.html#logging-advanced-tutorial 
[disable-not-public]: https://bugs.python.org/issue36318
[pt1]: /2020/07/18/python-logging.html
[pt2]: /2020/07/19/python-logging-filterhandler.html
[pt3]: /2020/07/21/dcc-defaults.html