--- 
title: "bashrc perl module version tip"
date: "2009-01-09"
slug: "2009/01/09/bashrc-perl-module-version-tip"
Tags: []
---
I often need to quickly check the version of a perl module currently installed.  A while back I got tired of running:

    $ perl -MPOE::Filter -E 'say $POE::Filter::VERSION'
    1.2357

So I added a quick function to .bashrc:

``` bash
pm-vers () {
    perl -M$1 -e "print \$$1::VERSION, \"\n\""
}
```

Now I just run:
    $ pm-vers POE::Filter
    1.2357


