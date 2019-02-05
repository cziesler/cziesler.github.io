---
layout: post
title: "Verilator"
date: 2019-02-04
categories:
  Verilog
  Verilator
---

[Verilator](https://www.veripool.org/wiki/verilator) is a Verilog simulator that translates synthesizable Verilog to C++ or SystemC. That translated code can then be compiled with a C++/SystemC testbench, and run in lieu of a commercial simulator.

I've used Verilator in the past to compile a design into a DLL file. That library was called by Excel, and used to draw waveforms of what the design would do with various register settings. It turned out to be a great tool for generating accurate waveforms for both the user guide, and for the firmware team wishing to find the right settings.

In this example, we'll simulate a simple 4-bit counter. This counter resets to 0, and increments on each clock edge out of reset.

```verilog
module test (
  input wire clk,
  input wire rst_n,
  output reg [3:0] cnt
);

always @(posedge clk or negedge rst_n) begin
  if (!rst_n) begin
    cnt <= 4'h0;
  end
  else begin
    cnt <= cnt + 4'h1;
  end
end

endmodule
```

The Verilator testbench file looks very similar to a traditional Verilog testbench -- instantiate the design under test, enable waveform dumping, initialize the variables, and run the simulation.

```cpp
#include "Vtest.h"
#include "verilated.h"
#include "verilated_vcd_c.h"

int main(int argc, char **argv, char **env) {
  Verilated::commandArgs(argc, argv);

  // init top verilog instance
  Vtest* top = new Vtest;

  // init trace dump
  Verilated::traceEverOn(true);
  VerilatedVcdC* tfp = new VerilatedVcdC;
  top->trace(tfp, 99);
  tfp->open("counter.vcd");

  // initialize simulation inputs
  top->clk   = 1;
  top->rst_n = 0;

  // run simulation
  for (int i = 0; i < 20; i++) {
    top->rst_n = !(i < 2);
    // dump variables into VCD file and toggle clock
    for (int clk = 0; clk < 2; clk++) {
      tfp->dump(2*i+clk);
      top->clk = !top->clk;
      top->eval();
    }
    printf("count: $%0X\n", top->cnt);
    if (Verilated::gotFinish()) exit(0);
  }

  tfp->close();
  exit(0);
}
```

To compile into C++, first call `verilator` with the Verilog module and the C++ testbench. The `-Wall` option enables all warnings. The `--cc` option creates a C++ output, rather than SystemC. The `--trace` option enables waveforms. The `--exe` option enables creating an executable. The `test.v` input is the Verilog module, and the `test_tb.cpp` input is the C++ testbench. The output of `verilator` will be a set of files, including a Makefile to compile the whole kit and caboodle into an executable (`obj_dir/Vtest.mk`).

```make
Vtest:
	verilator -Wall --cc --trace test.v --exe test_tb.cpp
	make -j -C obj_dir/ -f Vtest.mk Vtest
```

The executable can then be run like any traditional C++ program. Print statements will be displayed to the terminal, and a waveform will be created (counter.vcd).

```sh
% obj_dir/Vtest
count: $0
count: $0
count: $1
count: $2
count: $3
count: $4
count: $5
count: $6
count: $7
count: $8
count: $9
count: $A
count: $B
count: $C
count: $D
count: $E
count: $F
count: $0
count: $1
count: $2
```

This example can be extended to do many things. We could add some self-checking like a real testbench. We could capture the I/O and display them in a fancy [GUI](http://8bitworkshop.com/v3.3.0/?platform=verilog). We could even wire in keyboard inputs and create a GUI to create a [game](https://stackoverflow.com/a/38174654).

Verilator is an amazing tool with unlimited possibilities. I look forward to using it on my next project.

This example can be found on [GitHub](https://github.com/cziesler/verilator-test).
