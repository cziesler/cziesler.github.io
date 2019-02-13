---
layout: post
title: "Perk/Tk"
date: 2019-02-13
categories:
  Perl
  GUI
---

Whenever I want to create a GUI, I normally go to [Perl/Tk](https://metacpan.org/pod/Tk). This toolkit if full of widgets to make a full-fledged GUI.

[Tk](https://en.wikipedia.org/wiki/Tk_(software)) was developed in the early 1990s as an extension for Tcl. However, Tk can be used by more than just Tcl -- there are bindings for at least Haskell, Python, Ruby, Lisp, and most importantly Perl. Within Perl, there are several libraries -- [Tkx](https://metacpan.org/pod/Tkx), [Tcl::Tk](https://metacpan.org/pod/Tcl::Tk), and [Perl/Tk](https://metacpan.org/pod/Tk). I prefer Perl/Tk, mainly because I started with it and there are some great [books](https://www.amazon.com/Mastering-Perl-Tk-Graphical-Interfaces/dp/1565927168) on the topic.

Below is a bare bones example.

```perl
use Tk;
my $mw = MainWindow->new();
$mw->Label(-text=>"Hello, world")->pack();
MainLoop;
```

This will generate something like the following.

![Simple GUI](/assets/perk-tk/perl-tk-simple.png)

Let's break it down.

After including the library (`use Tk`), the `MainWindow->new()` call will create the main window. The [MainWindow](https://metacpan.org/pod/distribution/Tk/pod/MainWindow.pod) is a [Toplevel](https://metacpan.org/pod/Tk::Toplevel) widget that does not have a Parent (`$mw->Parent == undef`). Since the MainWindow is a type of Toplevel, we can pass in any of the Toplevel options, and in turn any [Widget](https://metacpan.org/pod/distribution/Tk/pod/Widget.pod) options.

```perl
use Tk;
my $mw = MainWindow->new(
  -title => "My First GUI",
);
$mw->Label(-text=>"Hello, world")->pack();
MainLoop;
```

![GUI With Title](/assets/perk-tk/perl-tk-title.png)

[Menubars](https://metacpan.org/pod/distribution/Tk/pod/Menu.pod) are a typical part of any GUI. To add one, simply create the Menu widget, and add it to the MainWindow. I like to follow the example in Chapter 12.2.2 in _Mastering Perl/TK_.

```perl
use Tk;
my $mw = MainWindow->new(
  -title => "My First GUI",
);
$mw->configure(
  -menu => $mw->Menu(
    -menuitems => [
      map [q/cascade/, $_->[0], -menuitems => $_->[1]],
        ['~File',
          [
            [qw/command ~Quit -accelerator Ctrl-q -command/ => \&exit],
          ],
        ],
    ],
  ),
);
$mw->Label(-text=>"Hello, world")->pack();
MainLoop;
```

Now notice that the accelerator option for Quit is set to "Ctrl-q". Unfortunately, this won't do anything on it's own -- we have to add some hotkeys. This is done by [binding to an event](https://metacpan.org/pod/distribution/Tk/pod/bind.pod). The first argument is a string containing the event (CTRL-Q in this case), and the second argument is a reference to the subroutine.

```perl
use Tk;
my $mw = MainWindow->new(
  -title => "My First GUI",
);
$mw->configure(
  -menu => $mw->Menu(
    -menuitems => [
      map [q/cascade/, $_->[0], -menuitems => $_->[1]],
        ['~File',
          [
            [qw/command ~Quit -accelerator Ctrl-q -command/ => \&exit],
          ],
        ],
    ],
  ),
);
$mw->bind("<Control-Key-q>", \&exit);
$mw->Label(-text=>"Hello, world")->pack();
MainLoop;
```

One last thing that I like to add to my GUIs is the ability to run an external script and capture the output to a text box. I normally use a [ROText](https://metacpan.org/pod/distribution/Tk/pod/ROText.pod) widget, since this ensures the captured text can't be modified. I also need to add a `require Tk::ROText` to avoid a warning.

I'll also add an [Entry](https://metacpan.org/pod/distribution/Tk/pod/Entry.pod) widget so that we can execute some commands.

```perl
use Tk;
require Tk::ROText;
my $mw = MainWindow->new(
  -title => "My First GUI",
);
$mw->configure(
  -menu => $mw->Menu(
    -menuitems => [
      map [q/cascade/, $_->[0], -menuitems => $_->[1]],
        ['~File',
          [
            [qw/command ~Quit -accelerator Ctrl-q -command/ => \&exit],
          ],
        ],
    ],
  ),
);
$mw->bind("<Control-Key-q>", \&exit);
my $t = $mw->ROText()->pack();
my $c = $mw->Entry(-textvariable => my $command)->pack();
MainLoop;
```

