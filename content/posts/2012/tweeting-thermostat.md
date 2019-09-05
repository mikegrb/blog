--- 
title: "tweeting thermostat"
date: "2012-05-20"
slug: "2012/05/20/tweeting-thermostat"
---
<img src="/images/2012/3m-50.jpg" />

I recently replaced the thermostat in our new house with a [Filtrete 3M-50](http://www.homedepot.com/buy/electrical-home-automation-security-home-automation-climate-control/filtrete-7-day-touchscreen-wifi-enabled-programmable-thermostat-with-backlight-182800.html).  This thermostat has a wifi module and is available at your local Home Depot for $99.  I'd been aware of it for a while but had planned on purchasing the $250 [nest](http://www.nest.com/).  The turning point for me came one even as I was going to bed.  I was hot, the heat was running and I chose to turn the fan on instead of going downstairs and adjusting the thermostat.  In my defense, I knew the thermostat would switch to night mode in another 45 minutes and reduce the set point but I was still wasting energy.

Comparing the Filtrete to the nest the main features you are missing out on are:

<!-- more -->

* automatic occupancy detection
* humidity detection
* automatic schedule learning

Automatic scheduling is a big win for the average person but our schedule isn't static enough for this to be much benifit to us.  The location of our thermostat is about the best place for a thermostat, but not for occupancy detection so this feature isn't missed.  I'm a bit sad with the loss of humidity detection, the next model up Filtrete has it for $200 but it's not a deal breaker.

The real win for geeks with a WIFI thermostat isn't the iPhone or Android Apps, it's the ability to interface with it and do geeky things.  The Filtrete 3M-50 is a rebadged [Radio Thermostat Company of America CT30](http://www.radiothermostat.com/control.html).  Radio Thermostat has the API documentation hidden on their website in the [latest news section](http://www.radiothermostat.com/latestnews.html#advanced).  Scroll to the bottom of the page and click the 'I Accept' checkbox.  There is also a [3rd party wiki](http://central.isaroach.com/wiki/index.php/Main_Page) with API documentation and sample code.

I started with the [poller.pl](http://central.isaroach.com/wiki/index.php/Perl#Poller) script available on the wiki to go from nothing to something quickly.  There's a lack of error checking in it and the non CPANed Radio::Thermostat module available on that page but it gets you up and running quickly.

<img src="/images/2012/temperature-daily.png" />

After this, I wrote a simple script to tweet daily thermostat information.

<pre><code>
#!/usr/bin/perl 

use 5.010;
use strict;
use warnings;

use RRDs;
use DateTime;
use Net::Twitter;
use Mojo::UserAgent;
use List::Util 'sum';

my $runtime  = get_runtime_string();
my $averages = get_averages_string();

die unless $runtime && $averages;

my $tweet = "Yesterday $averages with $runtime.";
say $tweet;

my $consumer_key    = '';
my $consumer_secret = '';
my $token           = '';
my $token_secret    = '';

my $thermostat_url  = 'http://10.8.8.98/tstat/datalog';
my $rrd_path        = '/home/michael/srv/therm/poller/data/temperature.rrd';

my $nt = Net::Twitter-&gt;new(
    traits   =&gt; [qw/OAuth API::REST/],
    consumer_key        =&gt; $consumer_key,
    consumer_secret     =&gt; $consumer_secret,
    access_token        =&gt; $token,
    access_token_secret =&gt; $token_secret,
);

$nt-&gt;update($tweet);


sub get_runtime_string {
    my $ua = Mojo::UserAgent-&gt;new;
    my $res = $ua-&gt;get($thermostat_url)-&gt;res;

    die "Didn't get runtimes for yesterday" unless $res;

    my $data = $res-&gt;json;
    return build_runtime_string('heat', $data-&gt;{ yesterday } )
        || build_runtime_string('cool', $data-&gt;{ yesterday } )
        || "no heat or A/C used";
}

sub build_runtime_string {
    my ($mode, $data)  = @_;
    my $times = $data-&gt;{ $mode . '_runtime' };
    return unless $times-&gt;{ hour } || $times-&gt;{ minute };
    return sprintf '%sing runtime of %i:%02i',
        $mode, $times-&gt;{ hour }, $times-&gt;{ minute };
}

sub get_averages_string {
    my $yesterday = DateTime-&gt;now()-&gt;subtract( days =&gt; 1);

    my $start_of_day = DateTime-&gt;new(
        year    =&gt; $yesterday-&gt;year,
        month   =&gt; $yesterday-&gt;month,
        day     =&gt; $yesterday-&gt;day,
        hour   =&gt; 0,
        minute =&gt; 0,
        second =&gt; 0,
        time_zone =&gt; 'America/New_York'
    );

    my $end_of_day = $start_of_day-&gt;clone-&gt;add( hours =&gt; 24 );

    my ($start,$step,$names,$data) = RRDs::fetch (
        $rrd_path,
        'AVERAGE',
        '-s ' . $start_of_day-&gt;epoch,
        '-e ' . $end_of_day-&gt;epoch
    );

    if (my $error = RRDs::error) {
        die "Error reading RRD data: $error";
    }

    my (@ext_temp, @int_temp);
    for my $row (@$data) {
        next unless grep { $_ } @$row;
        my ($outside, $inside) = @$row[2, 5];
        push @ext_temp, $outside;
        push @int_temp, $inside;
    }

   return  sprintf "averaged %.1fF outside and %.1fF inside",
        average( \@ext_temp), average(\@int_temp);
}

sub average {
    my $list = shift;
    return sum(@$list) / @$list;
}
</code></pre>

Tweets look like:

* Yesterday averaged 59.5F outside and 67.9F inside with no heat or A/C used.
* Yesterday averaged 64.2F outside and 70.1F inside with heating runtime of 0:17.
* Yesterday averaged 58.9F outside and 69.0F inside with heating runtime of 0:31.

You can follow my house on Twitter as [@mikegrbs_house](https://twitter.com/#!/mikegrbs_house).
