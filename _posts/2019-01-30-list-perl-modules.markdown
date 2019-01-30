---
layout: post
title: "List Installed Perl Modules"
date: 2019-01-30
categories:
  Perl
---

What Perl modules are installed for a given Perl installation? Here's a neat one-liner from [Stack Overflow](https://stackoverflow.com/a/14599339).

```sh
/usr/local/bin/perl -MFile::Find=find -MFile::Spec::Functions -Tlwe \
  'find { wanted => sub { print canonpath $_ if /\.pm\z/ }, no_chdir => 1 }, @INC'
```

Let's break it down.

```sh
/usr/local/bin/perl
```

This is the path to the Perl binary. On my machine, it's installed under the `/usr/local/bin` directory. If you want to use the Perl installation on your `$PATH`, simply call `perl`.

```sh
-MFile::Find=find -MFile::Spec::Functions
```

These will execute `use 'MODULE' qw('sub');` before executing any code. In this case, we want the `find` subroutine in [`File::Find`](https://perldoc.perl.org/File/Find.html) and all of [`File::Spec::Functions`](https://perldoc.perl.org/File/Spec/Functions.html).

```sh
-Tlwe
````

The first switch (`-T`) enables [taint mode](https://perldoc.perl.org/perlsec.html). Essentially, Perl will execute certain security checks as it's running.

The second switch (`-l`) enables automatic line-ending processing, essentially adding a new line to each print statement.

The third switch (`-w`) enables warnings (as should always be done in Perl scripts).

The final switch (`-e`) enables executing the argument passed in on the command line.

```perl
find { wanted => sub { print canonpath $_ if /\.pm\z/ }, no_chdir => 1 }, @INC
```

Let's add some line breaks and some additional syntactic sugar.

```perl
find (
  {
    wanted => sub {
      print canonpath $_ if /\.pm\z/
    },
    no_chdir => 1
  },
  @INC
);
```

The [`find`](https://perldoc.perl.org/File/Find.html) subroutine was imported by the `-MFile::Find=find` option, while `wanted` and `no_chdir` are items in the `%options` input argument to `find`.

The `wanted` hash option points to a code reference that performs some task on each file and directory. In this case, each item in `@INC` has it's [`canonpath`](https://perldoc.perl.org/File/Spec/Functions.html) printed if the item ends with `.pm`. This filters out directories, and files that are not Perl modules.

The `no_chdir` hash option to `find` ensures that `chdir()` will not be called for each directory as `find` recurses through the tree.

Of course, [TMTOWTDI](https://en.wikipedia.org/wiki/There%27s_more_than_one_way_to_do_it) in Perl. Everyone has their own script or one-liner to list their installed Perl modules.

* [https://www.perlmonks.org/?node_id=795418](https://www.perlmonks.org/?node_id=795418)
* [https://stackoverflow.com/questions/115425/how-do-i-get-a-list-of-installed-cpan-modules](https://stackoverflow.com/questions/115425/how-do-i-get-a-list-of-installed-cpan-modules)
* [http://perldoc.perl.org/perlfaq3.html#How-do-I-find-which-modules-are-installed-on-my-system%3F](http://perldoc.perl.org/perlfaq3.html#How-do-I-find-which-modules-are-installed-on-my-system%3F)

Is this version better? Probably not, but it works for me.
