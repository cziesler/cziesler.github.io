---
layout: post
title: "Verilog Plusargs"
date: 2019-01-28
categories:
  Verilog
---
In Verilog, plusargs can be used to change the behavior of a program without recompiling. This can be great if there are a lot of tests to run with the same testbench.

There are two calls for plusargs in Verilog: `$test$plusargs` and `$value$plusargs`.

The first call, `$test$plusargs("string")`, returns a boolean value. If the plusarg given by `"string"` was passed into the program at runtime, the call returns true. Otherwise, the call returns false.

The second call, `$value$plusargs("string=%d", val)`, also returns a boolean value depending on if the plusarg given by `"string"` was passed in. It also will load the register given by val based on the format given by `%d` in this example. Here is the list of formats available:

| Format | Description |
|-------|--------|
| `%b` | read as binary value |
| `%o` | read as octal value |
| `%d` | read as decimal value |
| `%h` | read as hex value |
| `%e` | read as real value, exponential format |
| `%f` | read as real value, decimal format |
| `%g` | read as real value, shortest format |
| `%s` | read as character string |

Note that the `$value$plusarg`s call is available since Verilog-2001. Before that, Verilog only supported the `$test$plusargs` call.

Here's a simple example. Here, the `my_port` variable is set to `1234` by default, unless the `+port=` plusarg is passed in on the command line when running the executable.

```verilog
module test;
int my_port;

initial begin
  if (!$value$plusargs("port=%d", my_port)) begin
    my_port = 32'd1234;
  end

  $display("my_port = %-d", my_port);
end
endmodule
```

Here is the invocation for compiling the file (using VCS):

```sh
% vcs +v2k -sverilog test.sv
```

And here is the invocation for running said program (again, using VCS):

```sh
% simv
my_port = 1234
```

Now, when passing in the `+port=5678` plusarg, here is the invocation and results:

```sh
% simv +port=5678
my_port = 5678
```

As you can see, no recompilation is necessary.

Now, say that you need to read in an array of plusargs. This might be useful for creating a look-up table of plusargs.

```verilog
module test2;

// Associative array with default values
int plusargs[string] = '{"a":1, "b":2, "c":3};

initial begin

  get_plusarg("a");
  get_plusarg("b");
  get_plusarg("c");

  foreach (plusargs[s]) begin
    $display("plusargs[%s] = %-d", s, plusargs[s]);
  end

end

function void get_plusarg(string name);
  $value$plusargs({name, "=%d"}, plusargs[name]);
endfunction

endmodule
```

We can compile with the following:

```sh
% vcs +v2k -sverilog test2.sv
```

And run with the following:

```sh
% simv
plusargs[a] = 1
plusargs[b] = 2
plusargs[c] = 3
```

With plusargs:

```sh
% simv +a=12 +b=15
plusargs[a] = 12
plusargs[b] = 15
plusargs[c] = 3
```

References:

* [http://stackoverflow.com/questions/16361272/passing-string-variables-to-plusargs-in-system-verilog](http://stackoverflow.com/questions/16361272/passing-string-variables-to-plusargs-in-system-verilog)
* [http://www.sutherland-hdl.com/papers/2002-DesignCon-tutorial_using_Verilog-2001_part2.pdf](http://www.sutherland-hdl.com/papers/2002-DesignCon-tutorial_using_Verilog-2001_part2.pdf)
