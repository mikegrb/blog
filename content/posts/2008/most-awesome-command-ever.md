--- 
title: "most awesome command ever"
date: "2008-05-27"
slug: "2008/05/27/most-awesome-command-ever"
Tags: []
---
I have a Debian Sid box that I'd been putting off updating for a while so finally got around to doing it tonight.  I took the opportunity to do a bit of spring cleaning and purge old packages I installed to play with and forgot to purge or removed instead of purging.  While doing this I discovered an interestingly named package, <code>timeout</code>.

<code>timeout</code> is a dead simple command, everything you need to know about it you can get from the usage information:

```
[michael@orion(~)] timeout
usage: timeout [-signal] time command...
```

And an example:
```
[michael@orion(~)] timeout 5 cat
Timeout: aborting command ``cat'' with signal 9
Killed
[michael@orion(~)] 
```

How did I not know about this before?!?
