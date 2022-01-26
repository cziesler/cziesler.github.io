---
layout: post
title: "Perl Getopt::Long"
date: 2022-01-26
categories:
  Perl
  Getopt::Long
---

For Perl command line programs, [Getopt::Long](https://perldoc.perl.org/Getopt::Long) is the de facto standard. It is used to capture command line options for use within the program.

Getopt::Long will parse `@ARGV`, extracting the specified options and possible values. Options start with one or two dashes -- one dash being used primarily for short single-letter options and two dashes for long more explicit options.

```
# Short options
-v -q

# Long options
--verbose --quiet
```

Command line options can also take one or more values.

```
# Single value
--opt value

# Multiple values
--color red --color blue
```

Here is a bare bones example. The variables (`$verbose`, `$quiet`, `$opt`, `@colors`) are declared beforehand, and are given default values. If a command line option is not passed in, the default value will be used. A reference to the variables are then passed into the GetOptions hash as the value, where the key is a string specifying the option name and optional values.

Notice that both `-v` and `--verbose` will set `$verbose`, since they are synonyms.

```perl
use Getopt::Long;
my $verbose = 0;
my $quiet = 0;
my $opt = "";
my @colors = ();
GetOptions (
  "v|verbose" => \$verbose,
  "q|quiet"   => \$quiet,
  "opt=s"     => \$opt,
  "color=s"   => \@colors,
);
print "v:$verbose, q:$quiet, opt:$opt, color:@colors\n";
```

Running this script with the options above would result in the following.

```
➜  ./t1.pl -v -q --opt value --color red --color blue
v:1, q:1, opt:value, color:red blue
```

In the above, the variables are declared before use in GetOptions, and given default values. This can be shortened by declaring and setting the variables in-line with GetOptions. Notice that `$colors` is now an array reference (there is probably a way to declare an array here, though I'm not entirely sure how).

```perl
use Getopt::Long;
GetOptions (
  "v|verbose" => \(my $verbose = 0),
  "q|quiet"   => \(my $quiet = 0),
  "opt=s"     => \(my $opt = ""),
  "color=s@"  => \(my $colors = []),
);
print "v:$verbose, q:$quiet, opt:$opt, color:@$colors\n";
```

```
➜  ./t2.pl -v -q --opt value --color red --color blue
v:1, q:1, opt:value, color:red blue
```

It's also possible to consolidate all of the options into a single hash reference. This can be useful for keeping all of the options together, reducing the amount of global variables, and allowing for passing it between objects.

```perl
use Getopt::Long;
my $self;
GetOptions (
  "v|verbose" => \($self->{verbose} = 0),
  "q|quiet"   => \($self->{quiet} = 0),
  "opt=s"     => \($self->{opt} = ""),
  "color=s@"  => \@{($self->{colors} = [])},
);
print "v:$self->{verbose}, q:$self->{quiet}, opt:$self->{opt}, color:@{$self->{colors}}\n";
```

```
➜  ./t3.pl -v -q --opt value --color red --color blue
v:1, q:1, opt:value, color:red blue
```

There are many other features to GetOptions, such as [options with hash values](https://perldoc.perl.org/Getopt::Long#Options-with-hash-values), [storing options values in a hash](https://perldoc.perl.org/Getopt::Long#Storing-options-values-in-a-hash), and [calling subroutines when an options is used](https://perldoc.perl.org/Getopt::Long#User-defined-subroutines-to-handle-options). Combining these together makes it easy to use parse command line options with Perl.

* [Getopt::Long](https://perldoc.perl.org/Getopt::Long)
