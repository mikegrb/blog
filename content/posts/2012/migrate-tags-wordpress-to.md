--- 
title: "migrate tags wordpress to"
date: "2012-05-08"
slug: "2012/05/08/migrate-tags-wordpress-to"
---
I've migrated from Wordpress to Octopress and used the Jekyll [wordpress migrator](https://github.com/mojombo/jekyll/wiki/Blog-Migrations) to move my posts over.  Unfortunately, this doesn't preserve post tags.  The output looks like this:

``` yaml
--- 
title: "migrate tags wordpress to"
date: "2011-10-27"
slug: "2011/10/27/migrate-tags-wordpress-to"
---
Earlier this month I gave a talk about...
```

Having the wordpress post id means extracting the post tags from the db should be quite easy.  First we define our desired output:

``` yaml
--- 
title: "migrate tags wordpress to"
date: "2011-10-27"
slug: "2011/10/27/migrate-tags-wordpress-to"
Tags: []
---
Earlier this month I gave a talk about ...
```
The tags field really can appear anywhere in this YAML fragment but I chose to throw it at the end.  With 103 posts to loop over, run a query and insert a new line a short script makes sense.  The real win for our script though is using the [Tie::File](https://metacpan.org/module/Tie::File) module which presents each file as an array with an element for each line.

<!-- more -->

From there it's simply a matter of reverse engineering the Wordpress schema to come up with a query that will return a space delimited list of tags for a given post id.

``` sql
SELECT GROUP_CONCAT(t.`name` SEPARATOR " ") FROM `wp_term_relationships` r
INNER JOIN `wp_term_taxonomy` tax ON r.`term_taxonomy_id` = tax.`term_taxonomy_id`
INNER JOIN `wp_terms` t ON tax.`term_id` = t.`term_id`
WHERE r.`object_id` = ?
AND tax.`taxonomy` = "post_tag";
```

Throw that query in a script that iterates over the list of files and uses [Tie::File](https://metacpan.org/module/Tie::File) to add a line to the post and we get:

<pre><code>
#!/usr/bin/perl 

use 5.010;
use strict;
use warnings;

use DBI;
use Tie::File;

my $path_to_posts = 'source/_posts';

my $dbh = DBI-&gt;connect('DBI:mysql:db_name:db_host','db_user','db_pass');

my $get_tags = $dbh-&gt;prepare(q{
    SELECT GROUP_CONCAT(t.`name` SEPARATOR " ") FROM `wp_term_relationships` r
    INNER JOIN `wp_term_taxonomy` tax ON r.`term_taxonomy_id` = tax.`term_taxonomy_id`
    INNER JOIN `wp_terms` t ON tax.`term_id` = t.`term_id`
    WHERE r.`object_id` = ?
    AND tax.`taxonomy` = "post_tag";
    });


for my $file (glob "$path_to_posts/*.markdown") {
    say $file;
    add_tags_to_file($file);
}

sub add_tags_to_file {
    my $filename = shift;
    tie my @file, 'Tie::File', $filename or die "No tie: $!";
    my $tags;
    while (my ($rec, $data) = each @file) {
        if ($data =~ m/^wordpress_id: (\d+)$/) {
            $tags = get_tag($1);
        }
        if( $data =~ /^date:/) {
            splice @file, $rec + 1, 0, $tags if $tags;
            return;
        }

    }
}

sub get_tag {
    my $post = shift;
    $get_tags-&gt;execute($post);
    my ($tags) = $get_tags-&gt;fetchrow_array();
    return "tags: " . $tags if $tags;
}

</code></pre>
