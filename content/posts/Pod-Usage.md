--- 
title: "Pod Usage"
date: "2014-06-08"
slug: "2014/06/08/Pod-Usage"
---

[Pod::Usage](https://metacpan.org/pod/Pod::Usage) is one of those boring modules that doesn't get the attention it deserves.  Included in Perl core since 5.6.0, released fourteen years ago, it's not a new kid on the block.  It's job isn't sexy, so not lots of talks, blog posts, or tweets about it. There are only 5 ++s for the distribution on [metacpan.org](https://metacpan.org/release/Pod-Usage). It's my goto module for all but the most trivial scripts, I probably *use* it second to only pragmas in scripts. (Haha, I made a funny.)

[Pod::Usage](https://metacpan.org/pod/Pod::Usage) generates usage/help output for your scripts by extracting relevant bits from your POD. This means you write usage/help information once for the POD and you get both your man page and -h command line help.  Let's look at its usage by starting with the result. I'll use [otfile](https://github.com/mikegrb/App-otfile/blob/master/script/otfile), a simple script for serving a file via HTTP from the command line for these examples.

```
$ otfile -h
Usage:
    $ otfile [-a -p 1234] <file to serve>

Options:
    --auto or -a
            Auto port selection. Increments specified port until successful.

    --port=N or -p N
            Use specified port, defaults to 1234.

    --multiple or -m
            Don't exit after serving the file the first time. To serve a
            file to multiple people. Requires, CTL+C to exit.

    --help or -h
            this help information

$
```

Nothing exceptional here, this is the style and content you would expect from any script on a POSIX like system. We get similarly unsurprising output for incorrect arguments.

```
$ otfile --invalid-arg-named-yaakov
Unknown option: invalid-arg-named-yaakov
Usage:
    $ otfile [-a -p 1234] <file to serve>

$
```

So what does the source look like? One of [Pod::Usage](https://metacpan.org/pod/Pod::Usage)'s benefits is the simplicity in using it.

```perl
use Pod::Usage;
use Getopt::Long;

GetOptions( ..., 'help' => sub { pod2usage(1) } ) or pod2usage(2);
```

That's it in terms of code! You can substitute Getopt::Std or your other argument parser of choice just as easily.  This was really cheating though since the real meat of it is in the POD.  So anything special there?

```perl
=head1 SYNOPSIS

$ otfile [-a -p 1234] <file to serve>

=head1 OPTIONS

=over 8

=item B<--auto> or B<-a>

Auto port selection.  Increments specified port until successful.

=item B<--port=N> or B<-p N>

Use specified port, defaults to 1234.

=item B<--multiple> or B<-m>

Don't exit after serving the file the first time. To serve a file to multiple
people. Requires, CTL+C to exit.

=item B<--help> or B<-h>

this help information

=back
```

Nothing special in the POD. [Pod::Usage](https://metacpan.org/pod/Pod::Usage) with the source I used outputs either the POD's SYNOPSIS section or the SYNOPSIS section and any section titled OPTIONS, ARGUMENTS or OPTIONS AND ARGUMENTS.

That's it!  Now, head over to [Pod-Usage on metacpan.org](https://metacpan.org/release/Pod-Usage) and give it some ++ love.
