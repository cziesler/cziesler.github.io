---
layout: post
title: "Reading an XLSX with Perl"
date: 2022-01-26
categories:
  Perl
  Spreadsheet
---

There have been many times where I needed to extract information from an Excel spreadsheet for use in other tools, such as SoC register maps, test cases, or even information from specification. For this, I go to [Spreadsheet::XLSX](https://metacpan.org/pod/Spreadsheet::XLSX). This Perl module provides functions for reading from an XLSX spreadsheet in Perl.

Spreadsheet::XLSX is an emulation of [Spreadsheet::ParseExcel](https://metacpan.org/pod/Spreadsheet::ParseExcel), meaning the important classes (Workbook, Worksheet, and Cell) are the same between the two modules -- just one parses XLSX and the other parses XLS. This makes it easy to swap between the two modules.

Below is a simple example that will open a spreadsheet _file.xlsx_, iterate through each sheet (_Sheet1_, _Sheet2_, and _Sheet3_), and print each cell of that sheet.

```perl
use Spreadsheet::XLSX;

# Open spreadsheet
my $excel = Spreadsheet::XLSX->new("file.xlsx");

# Go through each sheet
foreach my $sheet ($excel->worksheets()) {
  printf "Sheet: $sheet->{Name}\n";

  # Go through each cell
  foreach my $row ($sheet->{MinRow} .. $sheet->{MaxRow}) {
    foreach my $col ($sheet->{MinCol} .. $sheet->{MaxCol}) {
      my $cell = $sheet->get_cell($row, $col);
      my $val = ($cell) ? $cell->value() : "";
      print "[$val] ";
    }
    print "\n";
  }
}
```

Here are screenshots of the example _file.xlsx_.

![file.xlsx (Sheet1)](/assets/spreadsheet-xlsx/file-sheet1.png)
![file.xlsx (Sheet2)](/assets/spreadsheet-xlsx/file-sheet2.png)
![file.xlsx (Sheet3)](/assets/spreadsheet-xlsx/file-sheet3.png)

And here is the output of the script.

```
➜  ./t1.pl
Sheet: Sheet1
[Header1] [Header2]
[A] [1]
[B] [2]
[C] [3]
Sheet: Sheet2
[HeaderA] [HeaderB] [HeaderC]
[0] [t1] [12]
[1] [t2] [42]
[2] [t2] [3]
[3] [t4] []
[4] [t1] [2]
Sheet: Sheet3
[A] [B] [C]
[1] [2] [3]
[w] [] []
[] [x] []
```

It is important to check that a `$cell` exists before using its value, since a blank cell will cause `$cell` will be undefined.

```perl
my $val = ($cell) ? $cell->{Val} : "";
```

To switch between reading an XLSX and XLS, the following changes are needed:
* Change the module name from Spreadsheet::XLSX to Spreadsheet::ParseExcel.
* Change the file name to read in the XLS instead of the XLSX.
* ParseExcel requires an extra step of creating a Parser object, then parsing the spreadsheet with the `parse()` function.

```perl
use Spreadsheet::ParseExcel;

# Open spreadsheet
my $parser = Spreadsheet::ParseExcel->new();
my $excel  = $parser->parse("file.xls");

# Go through each sheet
foreach my $sheet ($excel->worksheets()) {
  printf "Sheet: $sheet->{Name}\n";

  # Go through each cell
  foreach my $row ($sheet->{MinRow} .. $sheet->{MaxRow}) {
    foreach my $col ($sheet->{MinCol} .. $sheet->{MaxCol}) {
      my $cell = $sheet->get_cell($row, $col);
      my $val = ($cell) ? $cell->value() : "";
      print "[$val] ";
    }
    print "\n";
  }
}
```

Or in other words, the diff between the two scripts is shown below.

```diff
5c5
< use Spreadsheet::XLSX;
---
> use Spreadsheet::ParseExcel;
8c8,9
< my $excel = Spreadsheet::XLSX->new("file.xlsx");
---
> my $parser = Spreadsheet::ParseExcel->new();
> my $excel  = $parser->parse("file.xls");
```

* [Spreadsheet::XLSX](https://metacpan.org/pod/Spreadsheet::XLSX)
