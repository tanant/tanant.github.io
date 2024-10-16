---
layout: post
title: "Python Qt Resource framework (.qrc)"
---
(Usual exercise, writing documentation and memory joggers for myself forces me to _actually_ make sure I remember/understand something)

Recently for RCAD I was building some interfaces in Qt (the standard VFX tooling and all), and I crossed the threshold where I needed to get some richer icons than what came out of the box. This means I need to get into the resource system and that's not hard, but it's something I don't use enough to have memorized.. so here's a brief reminder.

I'm using the stack of icons from [Qlementine](https://github.com/oclero/qlementine) for this example. Can't recall exactly why, but I think this suited the vibe I was going for..

## .qrc and resource compilation

- `.qrc` files are basically XML.

```
<!DOCTYPE RCC>
<RCC version="1.0">
  <qresource prefix="/qlementine/icons/32">
    <file>file/folder-open.svg</file>
  </qresource>
</RCC>
```

- The `qresource` prefix is what we'll use to refer to these things inside of the Qt context, the paths, obviously are relative to the .qrc file's path

_for python_ we do a compile step, using `rcc.exe` (they suggest using the correct pyside-rcc, which is fair.. but meh) which you can find in your Qt install's binary. 

`rcc.exe -o my_resource_glob.py -g python <all the .qrcs you want it to digest>`

which will output what is in effect an encoding of the data in binary strings... so _don't do this yourself and base64 encode icons_[^1]


## And in python?
Now in python, if you do an `import my_resource_glob` it should register this internally... and then you're left with the "now what?". Lemme explain in code, since you probably want to cut/paste anyhoo

#### Exploring the resource stack
{% highlight python %}
from Pyside2 import QtCore
# yeah, I use slightly deeper paths so over time I muscle-memorize the class structure

# treat the resource stack as a filesystem/hierarchy
# you can pass in a QtCore.QDir.Filter
files_in_my_d_drive = QtCore.QDir("d:/").entryList(QtCore.QDir.Files)
dirs_in_the_qrc_root = QtCore.QDir(":/").entryList(QtCore.QDir.Dirs)

# and from this example above, i'm using the qlementine icon set which is under qlementine/icons/<res>
for resolution in QtCore.QDir(":/qlementine/icons").entryList(QtCore.QDir.Dirs):   
    dirs = QtCore.QDir(f":/qlementine/icons/{resolution}").entryList(QtCore.QDir.Dirs)
    for dir in dirs:
        icon_count = len(QtCore.QDir(f":/qlementine/icons/{resolution}/{dir}").entryList(QtCore.QDir.Files))
        print(f"{dir}: {icon_count}")
{% endhighlight %}

#### Sticking an icon on a button
At this point, now you can access the icon and do something with it, like make a clap button
![a very boring clap button](/assets/images/qt_clap_button.png)
{% highlight python %}
icon = QtGui.QIcon(":/qlementine/icons/16/instruments/clap.svg")
pixmap = icon.pixmap(QtCore.QSize(256,256)) # pixmap sizing is totally another day's post :P
button = QtWidgets.QPushButton()
button.setIcon(pixmap)
button.setIconSize(QtCore.QSize(128,128))
button.show()
{% endhighlight %}

### Reload? Maaaybe not..

Something you might be doing as part of your python workflow is an `importlib.reload()` right? Just make sure you don't do it with this psuedo module. At the bottom if you dive in, you'll notice this bootstrapping:
{% highlight python %}
def qInitResources():
    QtCore.qRegisterResourceData(0x03, qt_resource_struct, qt_resource_name, qt_resource_data)

def qCleanupResources():
    QtCore.qUnregisterResourceData(0x03, qt_resource_struct, qt_resource_name, qt_resource_data)

qInitResources()
{% endhighlight %}

Notice that the import is used to run the `qInitResource` - if you reload the module you'll end up a bit of a messy situation from Qt. It may not explode immediately, but yeeeeeeeeaaaah, I don't think this is anything you want to explore in production (but to blow stuff up, sure!)


[^1]: I'm looking at you ChatGPT 😑
