---
layout: post
title: "Perl Objects"
date: 2022-03-08
categories:
  Perl
  Object-oriented
---

The following post shows how I create objects in Perl. As with all things in Perl [TMTOWTDI](https://en.wikipedia.org/wiki/There%27s_more_than_one_way_to_do_it) -- this is just my preferred way.

An object in Perl is normally a data structure such as a hash that has been explicitly associated with a particular class via the [bless](https://perldoc.perl.org/functions/bless) function in the constructor.

For example, the following `OO1` class uses `new` as the constructor (the constructor method name can be anything). The constructor expects the class name as the first argument (which Perl takes care of when it sees the `OO1->new()` method call).

I like to use named arguments, which I do through an array passed into the constructor (`quiet => 1`). By using named arguments, I'm able to give default values to arguments that aren't passed in -- including scalars, arrays, hashes, references, and more. I check for existence with [`exists`](https://perldoc.perl.org/functions/exists) -- when an argument is not passed in, the default is used. I use `exists` rather than the traditional "defined-or" `||` operator to support cases where `undef` is passed in as an argument.

In the constructor, there is an explicit conversion between an array and a hash when pulling arguments from `@_`. This avoids needing to pass a hash into the constructor (`OO1->new({quiet => 1})`).

At the end, the constructor needs to return a reference to the new object -- which is created by the `bless` function.

```perl
package OO1;

sub new {
  my ($class, %args) = @_;
  return bless {
    quiet => exists $args{quiet} ? $args{quiet} : 0,
    paths => exists $args{paths} ? $args{paths} : [],
    names => exists $args{names} ? $args{names} : {},
  }, $class;
}

1;
```

When using the class, I first need to tell Perl where to find the package. This is done by the [`use lib`](https://perldoc.perl.org/lib) directive, along with [`FindBin`](https://perldoc.perl.org/FindBin) avoiding hardcoded paths.

```perl
#!/usr/bin/perl

use strict;
use warnings;
use Data::Dumper;

# For Perl 5.26+, the cwd is not in the @INC search path.
use FindBin qw($RealBin);
use lib $RealBin;
use OO1;

my $oo1 = OO1->new();
print Dumper $oo1;

$oo1 = OO1->new(quiet => 1);
print Dumper $oo1;
```

Below is the result of running the test script. The first corresponds to the default values, and the second to when `quiet` is set to 1.

Notice that both objects are blessed and show the name of the class as the second argument to `bless`.

```
➜ ./t_oo1.pl
$VAR1 = bless( {
                 'quiet' => 0,
                 'names' => {},
                 'paths' => []
               }, 'OO1' );
$VAR1 = bless( {
                 'paths' => [],
                 'quiet' => 1,
                 'names' => {}
               }, 'OO1' );
```

Methods are be called on an object through the arrow operator (`->`).

The thing on the left side of the arrow is passed as the first argument to the subroutine -- in this case, the `$oo1` object. The other arguments are passed in afterwards.

```perl
my $oo1 = OO1->new();
$oo1->load("file");
```

The subroutine can use the object itself within the subroutine, such as to call additional methods, or to use the internal state.

```perl
sub load {
  my ($self, $file) = @_;
  $self->_parse($file);
}
```

## References

* [Object-Oriented Programming in Perl Tutorial](https://perldoc.perl.org/perlootut)
* [Perl Object Reference](https://perldoc.perl.org/perlobj)
* [Object Oriented Perl](http://perltraining.com.au/notes/perloo.pdf)
