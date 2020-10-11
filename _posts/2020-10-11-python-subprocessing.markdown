---
layout: post
title:  "A mini-dive into suprocessing in Python"
---
We've all been there, trying to get a python subprocess working and while there's many ways to make it work, generally the hacks I've seen (around the shell=True flag) stem from not knowing some of the gory details. I was on a PR at work commenting and saw this so thought I'd break this out a bit more for future reference.

Here's our basic setup:

{% highlight python %}
>>> import subprocess
>>> result = subprocess.check_call(args, ..., shell=??)  # because you ARE checking the call, right?
{% endhighlight %}

And from the docs args are..:

> ....required for all calls and should be a string, or a sequence of program arguments. Providing a sequence of arguments is generally preferred, as it allows the module to take care of any required escaping and quoting of arguments (e.g. to permit spaces in file names). If passing a single string, either shell must be True (see below) or else the string must simply name the program to be executed without specifying any arguments[^doc_ref]

## Shell yes? no? like, wha?
Shell = True/False has a major impact on what happens under the hood, so here's an exercise to try explain it all. Start by setting up something that's a proxy for a lot of what we'd expect to do - write some kind of 'executable' that we want to launch from python.

Create a file in your home dir called `foo.sh` that is 777 executable, and has this in it and only this (no shebang!)
{% highlight shell %}
echo $0 and $1 and $2 NOTHINGELSEISAFTERTHIS
{% endhighlight %}
Run it at the cli with up to two args ($0 is arg 0, $1 is arg 2 etc), here's what it does:
{% highlight shell %}
% foo.sh hello world now
./foo.sh and hello and world NOTHINGELSEISAFTERTHIS
{% endhighlight %}


### Breaking it once in python
Now in python, invoke this using the shell=False method:
{% highlight python %}
>>> retcode = subprocess.check_call(["/home/tanant/foo.sh", "what", "flavour", "pie"], shell=False)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/lib/python2.7/subprocess.py", line 185, in check_call
    retcode = call(*popenargs, **kwargs)
  File "/usr/lib/python2.7/subprocess.py", line 172, in call
    return Popen(*popenargs, **kwargs).wait()
  File "/usr/lib/python2.7/subprocess.py", line 394, in __init__
    errread, errwrite)
  File "/usr/lib/python2.7/subprocess.py", line 1047, in _execute_child
    raise child_exception
OSError: [Errno 8] Exec format error
{% endhighlight %}

This fails, because we're looking for an executable `/home/tanant/foo.sh`. This is not an executable. This is a text file with executable permissions which is interpreted by your shell. Since you're not using the shell, then this will fail.

This is also why the docs say that the string must simply name the program to be executed and have no args - programs can (but please don't) have spaces in their names...

### shell=True and command rewriting
Naturally, the first reaction is to swap in shell=True. 
{% highlight python %}
>>> retcode = subprocess.check_call(["/home/tanant/foo.sh", "what", "flavour", "pie"], shell=True)
/home/tanant/foo.sh and and NOTHINGELSEISAFTERTHIS
{% endhighlight %}

Hmm. Well. It ran. But it did't run like what we expected - notice almost all the args are buried? It's because this is doing `/bin/sh -c args[0] args[1]...` where args[0] happens to be the path to your text file. Also note that it's `/bin/sh` that gets multple args passed to it, not your command which explains the behaviour we see.

This is equivalent at your command line roughly to doing this: 
- (in the shell) `/bin/sh -c /home/tanant/foo.sh what flavour pie`;
- (in python) `subprocess.check_call(["/bin/sh", "-c", "/home/tanant/foo.sh", "what", "flavour", "pie"], shell=False)`

and these are almost certainly not what you want, it's only just a bit more obvious when you write the command out explicitly.

### Why not just pass a string?
In a more sane world, you would inclue the shebang in your file, so it looks like this: 
{% highlight shell %}
#!/bin/env bash
echo $0 and $1 and $2 NOTHINGELSEISAFTERTHIS
{% endhighlight %}

and often I'll see this as a hack:
{% highlight python %}
>>> retcode = subprocess.check_call(" ".join(["/home/tanant/foo.sh", "what", "flavour", "pie"]), shell=True)
/home/tanant/foo.sh and what and flavour NOTHINGELSEISAFTERTHIS
{% endhighlight %}

**Do not do this**. This is equivalent to passing the string `"/home/tanant/foo.sh what flavour pie"` as argument[0] to `/bin/sh`. 

Is that what you wanted? Probably not.

Why? Because I'm guessing you didn't want to allow the shell to decide how to break your string up into arguments. it just _happens_ to break up on spaces, but often what you pass in as arguments to your command already are preformatted and wrangled in python, and you don't want to get into the mess of shlexing and escaping words everywhere:

{% highlight python %}
>>> retcode = subprocess.check_call(["/home/tanant/foo.sh", "this is the first sentence", "another sentence", "pie"], shell=False)
/home/tanant/foo.sh and this is the first sentence and another sentence NOTHINGELSEISAFTERTHIS
{% endhighlight %}

{% highlight python %}
>>> retcode = subprocess.check_call(" ".join(["/home/tanant/foo.sh", "this is the first sentence", "another sentence", "pie"]), shell=True)
/home/tanant/foo.sh and this and is NOTHINGELSEISAFTERTHIS
{% endhighlight %}
------
[^doc_ref]: [Python 2, Section 17.1.1.1](https://docs.python.org/2/library/subprocess.html#frequently-used-arguments) docs 