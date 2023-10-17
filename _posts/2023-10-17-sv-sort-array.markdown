---
layout: post
title: "Sorting arrays in SystemVerilog"
date: 2023-10-17
categories:
  SystemVerilog
  Sorting
---

Today, I ran across a problem where I had to compare two arrays, but the order did not matter. To further complicate things, each array was made up of structs.

I figured the easiest way to compare the two arrays was to use the built-in `sort` method (*7.12.2 Array ordering methods* in IEEE 1800-2012) to sort both arrays, then compare them one-by-one. The `sort` method can be used on any unpacked array except for associative arrays, such as dynamic arrays and queues.

```verilog
int s[] = '{9, 1, 7};
s.sort; // s = {1, 7, 9}

string a[$] = '{"Hello", "world", "how", "are", "you?"};
a.sort; // a = {"Hello", "are", "how", "world", "you?"}
```

Of course, with a struct it gets a little trickier.

```verilog
typedef struct {
  int a;
  string n;
} t_pkt;

t_pkt x [] = '{
  '{42, "name 0"},
  '{7,  "name 1"},
  '{0,  "name 2"}
};
initial x.sort;
```

Compiling this with vcs gives me an error message:

```
Error-[SV-MWC] Missing with clause
testbench.sv, 27
tb, "x.sort;"
  Array method 'sort' requires a with clause as it cannot be applied directly 
  to the element type.
  Object: 'x' of type: t_pkt
  Please refer to the SystemVerilog LRM (1800-2012) Section 7.12 "Array 
  manipulation methods".
```

So to sort the array of structs, we need to use the `with` clause. The expression within `with` tells `sort` how to sort the array. By default, the iterator name is `item`.

```verilog
initial x.sort with (item.a); // x = '{'{0, "name 2"}, '{17, "name 1"}, '{42, "name 0"}}
```

More than one sorting condition can be used too. For example:

```verilog
typedef struct {
  int a, b;
} t_y;

t_y y [] = '{
  '{42, 0},
  '{7,  0},
  '{42, 2},
  '{7,  6},
  '{0,  1}
};
initial y.sort with ({item.a, item.b}); // '{'{a:0, b:1}, '{a:7, b:0}, '{a:7, b:6}, '{a:42, b:0}, '{a:42, b:2}} 
```

So back to the original problem -- comparing two arrays, ignoring order. This can be packaged up in a neat little function. Simply pass in two arrays (or queues), sort them both, check the sizes, then check the contents.

```verilog
function compare_arrays (t_y y0 [], t_y y1 []);
  y0.sort with ({item.a, item.b});
  y1.sort with ({item.a, item.b});

  if (y0.size != y1.size) return 0;

  foreach (y0[i]) begin
    if (y0[i].a != y1[i].a) return 0;
    if (y0[i].b != y1[i].b) return 0;
  end

  return 1;
endfunction

initial begin
  t_y y0 [] = '{'{0, 0}, '{1, 1}, '{1, 2}, '{3, 2}};
  t_y y1 [] = '{'{1, 1}, '{1, 2}, '{3, 2}, '{0, 0}};
  t_y y2 [] = '{'{1, 1},          '{3, 2}, '{0, 0}};
  t_y y3 [] = '{'{0, 0}, '{1, 1}, '{1, 2}, '{3, 0}};

  $display("y0 %s y0", compare_arrays(y0, y0) ? "==" : "!=");
  $display("y0 %s y1", compare_arrays(y0, y1) ? "==" : "!=");
  $display("y0 %s y2", compare_arrays(y0, y2) ? "==" : "!=");
  $display("y0 %s y3", compare_arrays(y0, y3) ? "==" : "!=");
end
```

Running this results in the expected output:

```
y0 == y0
y0 == y1
y0 != y2
y0 != y3
```

The code is published to EDA Playground: [Sorting arrays of structs](https://www.edaplayground.com/x/JwVL)
