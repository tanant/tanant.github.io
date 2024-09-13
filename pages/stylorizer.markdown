---
layout: default
title: Style Cheat Sheet
permalink: /stylomatic
---

This page just contains a longform example of most (all?) the styling i'm using so that I can easily regenerate things, swap themes around and check out what specific elements that I care about will look like.

# Heading 1 (general text elements)

Paragraph text will go here, inside we have [markdown link](links), [markdown_anchors][anchor] (see the bottom of the text). Note that the heading itself has an ID fragment that basically is replace all non-std chrs with - and lowercase. So you can anchor to those things.

Here's a list of 
- bullet
- points
- at
  - level 2
    - then three
       - then four
    - back to three

I also like ordered lists
1. they have a start
2. middle points
    1. and sublists as well are neat
        - sublists and bullets
            - sublists and bullets
2. and a clear end

We can ~~strikethrough~~ certain blocks of text


## Heading 2 (code fragments)

```
Triple quoted (fenced)
block of standard
Code text
```

Here is an `inline` backtick. Below is some tag-based highlighted javascript
{% highlight javascript %}
document.write("JavaScript is a simple language for javatpoint learners");
{% endhighlight %}

Below is some tag-based highlighted python with line nos
{% highlight python linenos %}
import foo
import bar

class Magic(object):
    def __init__():
        self.value = 3 * 2 + 1
        self.text = "text"
{% endhighlight %}

### Heading 3
Other less common things are say, footnotes[^cite1] and newer ones[^1]

#### Heading 4, other fancy elements

A horizontal ruler is either `---` or `***` on a line by itself and can divide content up (but must have a clear new line, or it looks like a heading!)

---



Definition Term (dt) and definition list (dl)
: are rendered like this


##### Heading 5
more text


#### Table of Contents

- [Underline](#underline)
- [Indent](#indent)
- [Center](#center)
- [Color](#color)

> **Warning:** Do not push the big red button.

> **Note:** Sunrises are beautiful.

> **Tip:** Remember to appreciate the little things in life.


[anchor]: foo
[^cite1]: this is a specific type of anchor
[^1]: more footnote