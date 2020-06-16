---
layout: post
title:  "WSL and making virtualenvs"
date:   2020-02-29 17:00:00 -0400
categories: stuff st
---

# TL;DR
Ugh. That was annoying. WSL and virtualenvs not playing nicely? Use metadata.

```
root@cuberdon:/mnt/c/Windows/system32# cat /etc/wsl.conf
[automount]
options = "metadata,uid=1000,gid=1000,umask=22,fmask=111"
```

### ...aaannd details
I've been progressively trying to set up my development environment properly using a mix of tools (I've gone down the somewhat silly route of just using whatever I have to hand so it's a.. well. It's a mess.) of which WSL is actually one of the core bits[^1]


For the longest time today I wasn't sure if I was going inane, or there was some secret sauce I was missing in order to make proper virtualenvs in WSL. This is what I was getting (I'm on Debian Buster)


{% highlight console %}
$ python3 -m venv ./playpen

The virtual environment was not created successfully because ensurepip is not
available.  On Debian/Ubuntu systems, you need to install the python3-venv
package using the following command.

    apt-get install python3-venv

You may need to use sudo with that command.  After installing the python3-venv
package, recreate your virtual environment.

Failing command: ['/mnt/e/venvs/foo/bin/python3', '-Im', 'ensurepip', '--upgrade', '--default-pip']
{% endhighlight %}

Reading through a heap of blogs and stuff, they always seem to come back to things being not installed or some other malarkey.. which in the WSL land is not the case. By way of demonstration, this works fine

{% highlight console %}
$ sudo python3 -m venv ./playpen
{% endhighlight %}

Which means there's something else in the works. You could just happily sudo forever (kinda) but it's been drilled into me that, just like becoming root, sudo is _not_ a normal practice and so since i'm not on the clock I spent some time killing zombies and letting my brain work out what was up... and turns out it's as i had an inkling, because everything was owned by root.

Odds on, if you've got a default setup in your WSL stack (or in my case, you're upgrading from an old WSL setup then you're going to be in an interesting state where your external drives are mounted thusly:

{% highlight console %}
C:\ on /mnt/c type drvfs (rw,noatime,uid=0,gid=0,case=off)
E:\ on /mnt/e type drvfs (rw,noatime,uid=0,gid=0,case=off)
{% endhighlight %}


This won't cause you any issues as you happily shove files around, but have you noticed that everything on C and E is owned by root? The 777/666 permissions means you might not notice it, but it's all owned by root.

What you *want to see* is this:

{% highlight console %}
C: on /mnt/c type drvfs (rw,relatime,metadata,case=off)
E: on /mnt/e type drvfs (rw,relatime,metadata,case=off)
{% endhighlight %}

*sigh*

Turns out there was a feature applied back in Jan 2018 to start mapping over metadata in WSL (Insider Build 17063+, but that's 2 years ago. You should have it by now), and did you know you can automatically configure this?

 - [Chmod/Chown WSL Improvements][chmod-chown]
 - [Automatically Configuring WSL][autoconf-wsl]

<br>

---
[^1]: I like playing games. Games tend to work on windows more than *nix. Is true. 


[chmod-chown]: https://devblogs.microsoft.com/commandline/chmod-chown-wsl-improvements/
[autoconf-wsl]: https://devblogs.microsoft.com/commandline/automatically-configuring-wsl/