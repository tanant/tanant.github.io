---
layout: post
title:  "About that char datatype.."
categories: c_datatypes code
---

So, I was helping [HanaSch][hanasch-gh] earlier today with some code around [CS50x, module 4], which is basically walking through a file, and learning about oldschool C style file pointers and suchlike, and I was roughly of the mind that the code that was provided as part of the module for the JFIF header detection was wonky.. turns out it's not, but that's not *entirely* the answer.

# File digestion in C
For a standard file read in, spit out kind of operation this is sort of what I've done for aaages now. It's been a while since I had to do file ops in C (don't find much use for files on an AVR...) but this is basically muscle memory these days:

{% highlight c %}
#define CHUNKSIZE 256
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    FILE *fp;
    int nread; // number of bytes just read
    char buffer[CHUNKSIZE];  // or char *buffer; buffer = malloc(sizeof(char)*512);
    fp = fopen(argv[1],"rb"); // FOR DEMO ONLY! Assume your users are malicious!
    while(!feof(fp)){
        nread = fread(buffer, sizeof(char), CHUNKSIZE, fp);
        // ... do *real* work stuff ...
        // debugging stuff here now
        printf("read: %04db file pointer is currently at position: %d\n", nread, ftell(fp));
        printf("            file pointer *was* at position: %d\n", ftell(fp)-nread);
        printf("data found: %02x %02x %02x %02x   %02x %02x %02x %02x \n", buffer[0], buffer[1], buffer[2], buffer[3], buffer[4], buffer[5], buffer[6], buffer[7]);  
        return 9;  // in this case, the first 8 octets will do to demonstrate
    }
}
{% endhighlight %}

and compiled and invoked thusly on my little WSL/debian setup:

{% highlight console %}
$ gcc foo.c -o foo.out
{% endhighlight %}



# The error crystalises..

The 'do real work stuff' here placeholder should've had some JFIF header detection which relies (and it's true in the sample file that's being used) on the headers and data being aligned on 512 byte boundaries, and just naively looks for 0xFF 0xD8 0xFF (the SOI, see the [JPEG article on wikipedia][jfif-wikipedia]). There's nothing wrong with this, and by all damn rights it should work... but it didn't and so for step 1 I cracked open the file in a hex editor, and stripped down the problem to bare basics. Here's the output:

{% highlight console %}
tanant@cuberdon:~/filconv_demo$ ./foo.out cat.jpg
read: 0256b file pointer is currently at position: 256
            file pointer *was* at position: 0
data found: ffffffff ffffffd8 ffffffff ffffffe0   00 10 4a 46
{% endhighlight %}

Wait. What? What the _actual_ flip? Why and when did characters get to be wha? 

The first thing I did was to apply an ugly `0x000000ff` mask to get it working but that felt (and is) so very very wrong, that I wanted to know what I'm screwing up here and so I did some digging based on a gut feel about something being wrong with the `fread` call, which takes us to [IEEE Std 1003.1-2017][susv4-2018-pu], which has an identical example with the same basic pattern.

So. At this point, being thoroughly confused I took a brief break.. and then it hit me. 

**Unsigned**. FML. It's an unsigned char I need. 

It turns out that for the longest time, I've been making the assumption that char == unsigned char, and that at some point, that assumption broke. Allow me to demonstrate:

{% highlight console %}
$ gcc foo.c -o foo.out -funsigned-char
$ ./foo.out cat.jpg
read: 0256b file pointer is currently at position: 256
            file pointer *was* at position: 0
data found: ff d8 ff e0   00 10 4a 46
{% endhighlight %}

{% highlight console %}
$ gcc foo.c -o foo.out -fsigned-char
$ ./foo.out cat.jpg
read: 0256b file pointer is currently at position: 256
            file pointer *was* at position: 0
data found: ffffffff d8 ffffffff ffffffe0   00 10 4a 46
{% endhighlight %}

So yes. Use an `unsigned char` when you want one.


[hanasch-gh]: https://github.com/HanaSch
[jfif-wikipedia]: https://en.wikipedia.org/wiki/JPEG_File_Interchange_Format#File_format_structure
[susv4-2018-pu]: https://pubs.opengroup.org/onlinepubs/9699919799/download/index.html
[CS50x, module 4]: https://cs50.harvard.edu/x/2020/psets/4/recover/