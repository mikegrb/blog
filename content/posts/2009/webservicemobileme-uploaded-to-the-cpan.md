--- 
title: "webservicemobileme uploaded to the cpan"
date: "2009-07-26"
slug: "2009/07/26/webservicemobileme-uploaded-to-the-cpan"
Tags: []
---
Just a quick post to announce a new perl module I uploaded to the CPAN the other day.  It's still to be considered alpha quality as it is lacking error checking and other such necessary stuffs but is working for me.

Usage looks like:

``` perl
use WebService::MobileMe;

my $mme = WebService::MobileMe->new(
    username => 'yaakov', password => 'HUGELOVE' );
my $location = $mme->locate;
print < <"EOT";
    As of $location->{date}, $location->{time}, Yaakov was at
    $location->{latitude}, $location->{longitude} (plus or minus
    $location->{accuracy} meters).
EOT
$mme->sendMessage( message => 'Hi Yaakov!', alarm => 1 );
```

The online documentation is available via <a href="http://search.cpan.org/~mikegrb/WebService-MobileMe/lib/WebService/MobileMe.pm">search.cpan.org</a>.


 * Git Web: <a href="http://git.thegrebs.com/?p=WebService-MobileMe">http://git.thegrebs.com/?p=WebService-MobileMe</a>
 * Git Clone: http://git.thegrebs.com/git/WebService-MobileMe
