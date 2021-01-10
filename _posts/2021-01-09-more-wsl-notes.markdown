---
layout: post
title:  "More WSL/Win10 notes for myself (and some memory joggers)"
---

My objectives with WSL are slllighty different, in that I'm planning to use this as a slight sidestep to using unix platform tools without having to worry about all the usual jazz of running a VM that i used to do. Also, I'm using it as a way to just keep this blog uptodate and do local tests etc etc.

### When you break things?
I broke something in one of the various updates/moves/hacking/repathing things I did which meant that installing/uninstalling Debian from the MS Store didn't help me out.

I got a `The system cannot find the path specified` - I'm betting that this is because I corrupted some data somewhere and install/uninstall still thought things were in a non-existing location.

Solution:  `wsl --unreigster <yourdistroname>` (and clean up after yourself)

### Moving things elsewhere  (and a rudimentary 'snapshot')
WSL installs/shoves the files on your `User` area - specifically:

- `%USERPROFILE\AppData\Local\Packages\`

I'm using debian, so it's `TheDebianProject.DebianGNULinux...`

I don't want this here because reasons (and I'm also going to forget where this is. Repeatedly). I also would like to snapshot and break things and go oops, and restore (and poke around in etc) so i keep them VHDXs in known location. You can do this using the export feature and do something like this

```
wsl --export Debian "i:\WSL_instances\debian.snapshot.$(Get-Date -UFormat `"%d%m%Y`").tar"
```

### Files and metadata and other stuff

Files go here: `\\wsl$\<distro_id>\....`

If you forget, you can also just do `/mnt/c/Windows/explorer.exe ~`

Don't forget as well to set up metadata in your `/etc/wsl.conf` [^metadata]

When first installed, the unix username you provide will be added to the sudoers list.

### Software
- aptitude
- screen (you will probably need to finangle)
- python3
- build-essential,  strace
- git, ack, tig
- wget, curl
- man

This is kinda sorted as my standard 'here's a box, go nuts' level. I have a snapshot of this which i can just upgrade periodically or reset to if i care enough about it. I will always just add to this as i go, but this is enough for me to start off with 

### Misc QoL stuff
append this to `.bashrc`
{% highlight shell %}
# because i run multiple differing instances, this should stop me from 
# accidentally working in one window where i didn't expect. Hopefully
DISTRONAME=baseline
PS1="[$DISTRONAME]:$PS1"
{% endhighlight %}

append this to `.bash_aliases`
{% highlight shell %}
alias ll="ls -la" 
{% endhighlight %}

and do this to `.inputrc`
{% highlight shell %}
# include the system root inputrc as a baseline
$include /etc/inputrc

# mappings for "page up" and "page down" to step to the beginning/end
# of the history
"\e[5~": beginning-of-history
"\e[6~": end-of-history

# use up and down for history search instead of simple search
# (because you can achieve that with a blank line)
"\e[A": history-search-backward
"\e[B": history-search-forward
{% endhighlight %}

### X11 
For X, i use [VcXsrv](https://sourceforge.net/projects/vcxsrv/) - on the windows host side, which requires a little bit of mucking around with windows firewall.

The short story is that you want to check the advanced settings for your firewall, and set it so you allow traffic through.

then install `x11-apps` and `xterm` (for xeyes, mainly)

then for WSL2, you need to export the correct DISPLAY host, and do something like this:

`export DISPLAY=$(grep -m 1 nameserver /etc/resolv.conf | awk '{print $2}'):0.0`

The 0.0 will be dependent on how you've rigged your X11 server of course, but there's a decent write up here [https://github.com/cascadium/wsl-windows-toolbar-launcher#firewall-rules](https://github.com/cascadium/wsl-windows-toolbar-launcher#firewall-rules)

[^metadata]: [my earlier notes](http://www.greenworm.net/stuff/st/2020/02/29/wsl-and-virtualenvs.html)
