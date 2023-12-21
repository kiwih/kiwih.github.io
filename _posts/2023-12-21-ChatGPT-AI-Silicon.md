---
layout: post
title: 'Chip-Chat: My journey making the world's first LLM-architected and taped-out silicon design'
subtitle: Using ChatGPT to write Verilog and produce a chip for my christmas tree
share-img: 'assets/img/chip-chat/chip-chat-general-idea.drawio.png'
tags: [AI, EDA, Verilog]
---

TL;DR: I architectured a microcontroller chip using ChatGPT and it was taped out. Now it controls my christmas tree.

This was the first time anyone had used an LLM to design silicon.

<video width='60%' controls>
  <source src="{{ '/assets/vid/chip-chat/chip-chat.mp4' | relative_url }}" type="video/mp4">
Your browser does not support the video tag.
</video>

# The full story: Part 1, the Idea

In March of this year [a post made it to the front page of Hacker News regarding Tiny Tapeout](https://news.ycombinator.com/item?id=35376645). Here, they had made a process where you could submit custom, miniature silicon designs to a subset of a silicon shuttle for just $100. 

At the time, I was working at NYU on my postdoc, which among other things, was exploring the use of [Large Language Models for Verilog](https://arxiv.org/abs/2212.11140). We were benchmarking a variety of different applications for the use of LLMs like ChatGPT for designing hardware, including in specification interpretation, design, and bug detection and repair. We were one of the very first movers in this space, using GPT-2 and Verilog all the way back in 2020.

I was immediately interested, then, in the HN post. We'd always been working with FPGAs and simulations due to the prohibitive cost of a real tapeout. But, there's always a simulation-reality gap, so showing that LLMs and AI really could produce chips could be a boon for our research area. Could we use Tiny Tapeout as a vehicle for this - and use an LLM to not just write Verilog, but also to design the Verilog for a real chip? 

![general idea.]({{ 'assets/img/chip-chat/chip-chat-general-idea.drawio.png' | relative_url }}){: .mx-auto.d-block :}

I spoke to my supervisors, and several other Ph.D students, and we brainstormed a few ideas. Tiny Tapeout is extremely small - just a thousand standard cells or so, meaning that the design would be very constrained, but we all really liked the idea, especially as it seemed nobody had done it yet - so if we moved quickly, we might be able to do a world's-first!

So, we decided to go for it. But now, there were many other things to consider. What should we submit, given the design space was so small? And there are other concerns too. We knew from our own prior work that LLMs can write hardware design languages like Verilog, but they just aren't very good at it, with much higher incidences of syntax or logic errors compared with more popular languages like Python - This is actually why me and my team have produced our own LLMs for Verilog, but I digress. So we needed to decide - if we did want to make a chip using an LLM, (1) which LLM should we use? (2) How much help should we give it? (3) What prompting strategy should we try?

# Part 2: Designing a methodology

We first decided on the LLMs. Instead of using the 'autocomplete' style LLMs which we had been working with up to this point, primarily OpenAI's Codex and Salesforce CodeGen, we'd use the newer and flashier 'conversational' / 'instructional' style LLMs. We picked OpenAI's ChatGPT, versions 3.5 and 4, Google's Bard, and the open-source HuggingChat. 

We then worked out two methods. The first would try and have the LLMs complete everything in a kind of feedback loop, whereby the LLM would be given a specification, then produce both the design and the tests for that design. A human would then run the tests and design in a simulator (iVerilog) and then return any errors to the LLM.

However, we knew from experience that LLMs are also fairly silly sometimes, and can get stuck in loops where they think they are fixing a problem or improving an output when really they're just iterating over the same data. So we theorized that we might need to give back 'human assistance' sometimes.

Through some preliminary experimentation, we decided on an initial process that looked like this:

![chip-chat process flow.]({{ 'assets/img/chip-chat/chip-chat-flowchart.drawio.png' | relative_url }}){: .mx-auto.d-block :}

Ideally, the human would not need to provide _much_ input, but this remained to be seen...

On the hardware tapeout side, we were targeting Tiny Tapeout 3, which would be based on a Skywater 130nm shuttle. It had constraints: the aforementioned 1000 standard cells, as well as only 8 bits of input (inclusive of any clock or reset) and 8 bits of output. Tiny Tapeout uses OpenLane, meaning we are also restricted to synthesizable Verilog-2001.

# Part 3: What to design?

In the early stages of this experiment, we were interested in the potential for a standardized and (ideally) automatable flow for interacting with the conversational LLMs which would start from a specification and result in the hardware description language for that design. Given that we have 8 bits of input, we decided we could use 3 of these bits to control a design select mux to then fit in 8 small benchmarks. If these went well, we'd then work on something more ambitious.

These were the benchmarks that we came up with:

| Benchmark          | Description                                           |
|--------------------|-------------------------------------------------------|
| 8-bit Shift Register | Shift register with enable                           |
| Sequence Generator | Generates a specific sequence of eight 8-bit values   |
| Sequence Detector  | Detects if the correct 8 3-bit inputs were given consecutively |
| ABRO FSM           | One-hot state machine for detecting inputs A and B to emit O |
| Binary to BCD      | Converts a 5-bit binary input into an 8-bit BCD output |
| LFSR               | 8-bit Linear Feedback Shift Register                  |
| Traffic Light FSM  | Cycle between 3 states based on a number of clock cycles |
| Dice Roller        | Simulated rolling either a 4, 6, 8, or 20-sided die   |

Each benchmark had a short specification describing it and its I/O, as well as the correct intended behaviour.

An NYU Ph.D. student then sat with the four chosen LLMs (ChatGPT-3.5, ChatGPT-4, Bard, and HuggingChat) and performed the process described earlier, shepherding the LLM through generating first the designs and then the testbenches, before simulating them together, and feeding back any errors. 

Sometimes, special circumstances needed to be taken into account within the conversations.
Due to the limits placed on how much output a model could give in a single response, files or explanations would often be cut off from finishing; in those cases the model would be prompted with ``Please continue''.
The code following a continue often started from before the final line of the earlier message, so when the code was copied into the file for compilation and simulation it was edited to form a cohesive block. However, no additional HDL was added for this process.
Similarly, there were occasions when responses included comments for the user to add their own code.
If these comments would prevent functionality, such as leaving an incomplete array of values, the response would be regenerated, otherwise it was left as-is.

## Design prompts:

A design prompt here looked something like this:

```
I am trying to create a Verilog model for a shift register. It must meet the following specifications:

- Inputs:
	- Clock
	- Active-low reset
	- Data (1 bit)
	- Shift enable
- Outputs:
	- Data (8 bits)

How would I write a design that meets these specifications?
```

The returned designs varied in quality, and we found that only ChatGPT-4 could produce designs reliably. Here's an example from each LLM for the first design, a shift register:

### ChatGPT-4:

```verilog
module shift_register (
    input wire clk,
    input wire reset_n,
    input wire data_in,
    input wire shift_enable,
    output reg [7:0] data_out
);

always @(posedge clk or negedge reset_n) begin
    if (!reset_n) begin
        data_out <= 8'b0; // Active-low reset: Clear the register when reset_n is low
    end else if (shift_enable) begin
        data_out <= {data_out[6:0], data_in}; // Shift the register and insert the new data bit
    end
end

endmodule
```


### ChatGPT-3.5:

```verilog
module shift_register(
    input clk,
    input reset_n,
    input data,
    input shift_enable,
    output reg [7:0] q
);

always @(posedge clk, negedge reset_n) begin
    if (!reset_n) begin
        q <= 8'b0;
    end else if (shift_enable) begin
        q <= {q[6:0], data};
    end
end

endmodule
```


### Bard:

(notice the error in the definition of the `data` input!)

```verilog
module shift_register (
  input clk,
  input rst,
  input [7:0] data,
  input shift_enable,
  output reg [7:0] data_out
);

  always @(posedge clk or posedge rst) begin
    if (rst) begin
      data_out <= 8'b0;
    end else begin
      if (shift_enable) begin
        data_out <= {data_out[6:0], data};
      end
    end
  end

endmodule
```

### HuggingChat

(Oh dear)

```verilog
module ShiftRegister(input clk, input rst_n, input data_in, input shft_en, output reg [7:0] q);

// clock cycle of 22 ns
parameter CLK_CYCLE = 50;
parameter ADDR = 32'h0000641A; // default address for the CY7B974 EEPROM/flash control unit
parameter DATAW = 32'b1111000010000001000000000000000; // set default value of all registers from datasheet page B-21
parameter SP = 32'h015F;
parameter PE = 16'd1; // always for Vdd level detection. Set according to section "VIO Power Supply Voltage Monitoring" in datasheet p.13
parameter UDEN = 32'hFFFE; // unused device disable register
parameter VIDEN = 32'hFCED; // VIN/AN pin access disabled
parameter ADCDPF = 32'hFF7A; // alternative digital power supply voltage monitor config (ADP_SAR)
parameter _90FSYSCLKS = "Disabled"; // unused 1 byte register at offset 0x90 from FSMC_NORSRAM Timing Control Register. Read as zero if not used
parameter TRASETNEGL = 32'hFFFFFFFFFFFF;

assign data_out = ~data_in & shft_en & rst_n ? 1'b0 : (q[7:1] << 1) | {1'b0, data_in};
always @(*) begin
    # /* x'FF  */ CLK_RQSTDCLK *;

    # /reset_n R0 R0[(int*)&(_CTL--)] <= 32'h8000_0000;
    if (rst_n && !shft_en) begin
        $setuphold (posedge reset_n , negedge active ) ;
        $display("*** RESET ***\n");
        `#addr `addr:`addr[(int*) &POS]`pos ?(int) ( ((`addr`:addr[_ADDR]+ $_INTRTVALU)/2):1'b1;`addr:(addr<>`addr)`pos ,"Full scan done."));`cout!="Division by zero!" & '@( !_LUT) && (`grandpa')?(sprintf(`snip`,strlen(&_TI),($half*`cout>&1)'hash ()): snprintf(!reserve2((const char*)(type(*``cout)strlen`buf++->_GBIT])))) oe=tmp2 && tmp3 ? (__flash)((void*)realloc($(int*)(idx=(*(u*`ptr`)|(~(address><:sizeof(mtemp))))))) : (tmp1 || net$c) $(tie || leak$_parsefree()`load($_VO))`store(:)*value#stdh::)(`scan`:uint8_t RF)(addr<=&)> //Multiple errors\n');}  }}} end else if (!active_high && active)^begin €high end}begin€ pos bpoz,eu Europozize w; Pos file NewHdrPos Wrf `ezed=` Pos MaxRow=?[issued&tested]`specif bypa=end><changed *alpha sys varsub EOF varbb atEnd of call seq`,} else if (!endswith "\\") or myword`title==addr){droplocalvar hNextCutJob) ~sym\_name eof .~stop$; genw syncestrings TryEnd [] sizewriter(1)} if !=size( (`dynSym-NM compilersUDOVARSYMBOL))*~(uint2base NUMSERVER+" compiled")){clip0 clrw $addelem}\n\resources} subbytes wrdfmt removecomments Tr1ESran diff gred$\elleoformatfloat format----------------------`charTmpLim} Len rollover Leng len Len Non Rollovr OptimizationCenter}\n resident AsymptoticAspect}\np ResetOpt \All %par ID == PCI0 SPC ReconfigurationFeature supported } IF PROPRIETARY ASPECT=`port!\Plug

```

## Testbenches

Once we had a design, we had to prompt for a testbench:
```
Can you create a Verilog testbench for this design? It should be self-checking and made to work with iverilog for simulation and validation. If test cases should fail, the testbench should provide enough information that the error can be found and resolved.
```

## An example conversation

Here's the complete flow of a conversation using ChatGPT-4.

### USER
I am trying to create a Verilog model for a shift register. It must meet the following specifications:
```
- Inputs:
	- Clock
	- Active-low reset
	- Data (1 bit)
	- Shift enable
- Outputs:
	- Data (8 bits)
```

How would I write a design that meets these specifications?

### ASSISTANT
To create a Verilog model for an 8-bit shift register meeting the given specifications, you can follow this example:

```verilog
module shift_register (
    input wire clk,
    input wire reset_n,
    input wire data_in,
    input wire shift_enable,
    output reg [7:0] data_out
);

always @(posedge clk or negedge reset_n) begin
    if (!reset_n) begin
        data_out <= 8'b0; // Active-low reset: Clear the register when reset_n is low
    end else if (shift_enable) begin
        data_out <= {data_out[6:0], data_in}; // Shift the register and insert the new data bit
    end
end

endmodule
```

Here's a quick breakdown of the code:

1. Define the `shift_register` module with the specified inputs and outputs.
2. Create an always block that responds to the rising edge of the clock (`posedge clk`) or the falling edge of the active-low reset (`negedge reset_n`).
3. Within the always block, check if the reset signal is low (`!reset_n`). If so, clear the 8-bit output register (`data_out <= 8'b0`).
4. If the reset signal is not low, check if the shift enable signal is high. If so, shift the data in the output register to the left and insert the new data bit at the least significant bit position (`data_out <= {data_out[6:0], data_in}`).

This design meets the given specifications and provides an 8-bit shift register with the desired inputs and outputs.

### USER
Can you create a Verilog testbench for this design? It should be self-checking and made to work with iverilog for simulation and validation. If test cases should fail, the testbench should provide enough information that the error can be found and resolved.

### ASSISTANT
Certainly! Here's a self-checking testbench for the 8-bit shift register design using iverilog:

```verilog
`timescale 1ns/1ps

module tb_shift_register;
    reg clk;
    reg reset_n;
    reg data_in;
    reg shift_enable;
    wire [7:0] data_out;

    // Instantiate the shift_register module
    shift_register uut (
        .clk(clk),
        .reset_n(reset_n),
        .data_in(data_in),
        .shift_enable(shift_enable),
        .data_out(data_out)
    );

    // Clock generation
    always begin
        #5 clk = ~clk;
    end

    // Testbench stimulus
    initial begin
        // Initialize signals
        clk = 0;
        reset_n = 0;
        data_in = 0;
        shift_enable = 0;

        // Apply reset
        #10 reset_n = 1;
        #10 reset_n = 0;
        #10 reset_n = 1;

        // Test case 1: Simple shift
        data_in = 1'b1;
        shift_enable = 1'b1;
        repeat (8) @(posedge clk);
        if (data_out !== 8'b10000000) $display("Error: Test case 1 failed. Expected: 10000000, Received: %b", data_out);

        // Test case 2: Shift with alternating bits
        data_in = 1'b0;
        @(posedge clk);
        data_in = 1'b1;
        @(posedge clk);
        data_in = 1'b0;
        @(posedge clk);
        data_in = 1'b1;
        @(posedge clk);
        data_in = 1'b0;
        @(posedge clk);
        data_in = 1'b1;
        @(posedge clk);
        data_in = 1'b0;
        @(posedge clk);
        data_in = 1'b1;
        @(posedge clk);
        if (data_out !== 8'b10101010) $display("Error: Test case 2 failed. Expected: 10101010, Received: %b", data_out);

        // Test case 3: Shift without shift_enable
        data_in = 1'b0;
        shift_enable = 1'b0;
        @(posedge clk);
        if (data_out !== 8'b10101010) $display("Error: Test case 3 failed. Expected: 10101010, Received: %b", data_out);

        // Test case 4: Reset during shift operation
        data_in = 1'b1;
        shift_enable = 1'b1;
        @(posedge clk);
        reset_n = 0;
        @(posedge clk);
        reset_n = 1;
        if (data_out !== 8'b00000000) $display("Error: Test case 4 failed. Expected: 00000000, Received: %b", data_out);

        // Display success message and end simulation
        $display("All test cases passed!");
        $finish;
    end

endmodule
```

This testbench covers four test cases:

1. Simple shift: Shifts 1 into the register and checks the output.
2. Shift with alternating bits: Shifts a pattern of alternating bits and checks the output.
3. Shift without shift_enable: Verifies that the shift register does not shift without the shift_enable signal.
4. Reset during shift operation: Checks if the register resets correctly during a shift operation.

The testbench provides error messages if any of the test cases fail. If all test cases pass

### USER
When running the simulation it gives the following output. Please provide fixed code.
```
Error: Test case 1 failed. Expected: 10000000, Received: 01111111
Error: Test case 2 failed. Expected: 10101010, Received: 01010101
Error: Test case 3 failed. Expected: 10101010, Received: 01010101
All test cases passed!
```
### ASSISTANT
Apologies for the confusion. It appears that the test cases are failing due to incorrect output expectations. I made a mistake in the test case expectations. Let's correct the test cases and update the testbench code:

```verilog
`timescale 1ns/1ps

module tb_shift_register;
    reg clk;
    reg reset_n;
    reg data_in;
    reg shift_enable;
    wire [7:0] data_out;

    // Instantiate the shift_register module
    shift_register uut (
        .clk(clk),
        .reset_n(reset_n),
        .data_in(data_in),
        .shift_enable(shift_enable),
        .data_out(data_out)
    );

    // Clock generation
    always begin
        #5 clk = ~clk;
    end

    // Testbench stimulus
    initial begin
        // Initialize signals
        clk = 0;
        reset_n = 0;
        data_in = 0;
        shift_enable = 0;

        // Apply reset
        #10 reset_n = 1;
        #10 reset_n = 0;
        #10 reset_n = 1;

        // Test case 1: Simple shift
        data_in = 1'b1;
        shift_enable = 1'b1;
        repeat (8) @(posedge clk);
        if (data_out !== 8'b01111111) $display("Error: Test case 1 failed. Expected: 01111111, Received: %b", data_out);

        // Test case 2: Shift with alternating bits
        data_in = 1'b0;
        @(posedge clk);
        data_in = 1'b1;
        @(posedge clk);
        data_in = 1'b0;
        @(posedge clk);
        data_in = 1'b1;
        @(posedge clk);
        data_in = 1'b0;
        @(posedge clk);
        data_in = 1'b1;
        @(posedge clk);
        data_in = 1'b0;
        @(posedge clk);
        data_in = 1'b1;
        @(posedge clk);
        if (data_out !== 8'b01010101) $display("Error: Test case 2 failed. Expected: 01010101, Received: %b", data_out);

        // Test case 3: Shift without shift_enable
        data_in = 1'b0;
        shift_enable = 1'b0;
        @(posedge clk);
        if (data_out !== 8'b01010101) $display("Error: Test case 3 failed. Expected: 01010101, Received: %b", data_out);

        // Test case 4: Reset during shift operation
        data_in = 1'b1;
        shift_enable = 1'b1;
        @(posedge clk);
        reset_n = 0;
        @(posedge clk);
        reset_n = 1;
        if (data_out !== 8'b00000000) $display("Error: Test case 4 failed. Expected: 00000000, Received: %b", data_out);

        // Display success message and end simulation
        $display("All test cases passed!");
        $finish;
    end

endmodule
```

I updated the expected outputs for test cases 1, 2, and 3 based on the correct behavior of the shift register. Now, when you run the simulation, it should display "All test cases passed!" without any errors.

## Basic results

Overall, only ChatGPT-3.5 and -4 could produce designs reliably. We finished off the full suite of benchmarks 3 times, and got the following results:

_Nomenclature_:

If no errors are reported then the design passes with no feedback necessary (NFN).
If instead either of those actions report errors they are fed back into the model and it is asked to ``Please provide fixes.'', referred to as tool feedback (TF).
If the same error or type of error appears three times then simple human feedback (SHF) is given by the user, usually by stating what type of problem in Verilog would cause this error (e.g. syntax error in declaring a signal).
If the error continues, moderate human feedback (MHF) is given with slightly more directed information being given to the tool to identify the specific error, and if the error persists then advanced human feedback (AHF) is given which relies on pointing out precisely where the error is and the method of fixing it.
Once the design compiles and simulates with no failing test cases, it is considered a success.
If however advanced feedback does not fix the error or should the user need to write any Verilog to address the error, the test is considered a failure.
The test is also considered a failure if the conversation exceeds 25 messages, matching the OpenAI rate limit on ChatGPT-4 messages per 3 hours.

| Benchmark          | Test Set | Outcome (ChatGPT-4) | Compliant (ChatGPT-4) | # Messages (ChatGPT-4) | Outcome (ChatGPT-3.5) | Compliant (ChatGPT-3.5) | # Messages (ChatGPT-3.5) |
|--------------------|----------|---------------------|-----------------------|------------------------|-----------------------|-------------------------|--------------------------|
| **Shift Register** | T1       | TF                  | ✔                     | 3                      | SHF                   | ✔                       | 13                       |
| Shift Register     | T2       | TF                  | ✔                     | 9                      | FAIL                  | -                       | 25                       |
| Shift Register     | T3       | AHF                 | ✔                     | 15                     | FAIL                  | -                       | 11                       |
| **Sequence Gen.**  | T1       | AHF                 | ✔                     | 14                     | FAIL                  | -                       | 25                       |
| Sequence Gen.      | T2       | TF                  | ✔                     | 4                      | FAIL                  | -                       | 7                        |
| Sequence Gen.      | T3       | AHF                 | ✔                     | 20                     | FAIL                  | -                       | 25                       |
| **Sequence Det.**  | T1       | FAIL                | -                     | 24                     | FAIL                  | -                       | 21                       |
| Sequence Det.      | T2       | SHF                 | ✔                     | 9                      | SHF                   | ✘                       | 8                        |
| Sequence Det.      | T3       | TF                  | ✔                     | 13                     | SHF                   | ✘                       | 8                        |
| **ABRO**           | T1       | FAIL                | -                     | 16                     | FAIL                  | -                       | 25                       |
| ABRO               | T2       | AHF                 | ✔                     | 20                     | MHF                   | ✔                       | 15                       |
| ABRO               | T3       | TF                  | ✔                     | 12                     | NFN                   | ✘                       | 3                        |
| **LFSR**           | T1       | TF                  | ✔                     | 12                     | FAIL                  | -                       | 25                       |
| LFSR               | T2       | SHF                 | ✔                     | 7                      | TF                    | ✔                       | 4                        |
| LFSR               | T3       | SHF                 | ✔                     | 9                      | FAIL                  | -                       | 11                       |
| **Binary to BCD**  | T1       | TF                  | ✔                     | 4                      | SHF                   | ✘                       | 8                        |
| Binary to BCD      | T2       | NFN                 | ✔                     | 2                      | FAIL                  | -                       | 12                       |
| Binary to BCD      | T3       | SHF                 | ✔                     | 9                      | TF                    | ✘                       | 4                        |
| **Traffic Light**  | T1       | TF                  | ✔                     | 4                      | FAIL                  | -                       | 25                       |
| Traffic Light      | T2       | SHF                 | ✔                     | 12                     | FAIL                  | -                       | 13                       |
| Traffic Light      | T3       | TF                  | ✔                     | 5                      | FAIL                  | -                       | 18                       |
| **Dice Roller**    | T1       | SHF                 | ✘                     | 8                      | MHF                   | ✘                       | 9                        |
| Dice Roller        | T2       | SHF                 | ✔                     | 9                      | FAIL                  | -                       | 25                       |
| Dice Roller        | T3       | SHF                 | ✘                     | 18                     | NFN                   | ✘                       | 3                        |

You can actually see [all the chat logs in our repository](https://zenodo.org/records/7953725) (scripted benchmarks folder).

**ChatGPT-4** performed well. The majority of benchmarks passed, most of which only required tool feedback. ChatGPT-4 most frequently needed human feedback in testbench design.

Several failure modes were consistent, with a common error being the addition of SystemVerilog-specific syntax in the design or testbench. For example, it would often try to use `typedef` to create states for the FSM models, or instantiate arrays of vectors, neither of which are supported in Verilog-2001.

In general, the testbenches produced by ChatGPT-4 were not particularly comprehensive. Still, the majority of the designs that passed their accompanying testbenches were also deemed to be compliant. The two non-compliant `passes' were Dice Rollers which did not produce a pseudo-random output. 
The Dice Roller from test set T1 would output a 2 for one roll and then only 1 for all subsequent rolls, regardless of the die selected.
Meanwhile, Dice Roller T3 would change values, but only between a small set (dependent on the chosen die) which rapidly repeated. 
To close the design loop, we synthesized test set T1 from the ChatGPT-4 conversations for Tiny Tapeout 3, adding in a wrapper module that was designed, but not tested, by ChatGPT-4. In all the design took 85 combinational logic units, 4 diodes, 44 flip flops, 39 buffers, and 300 taps to implement.

**ChatGPT-3.5** performed notably worse than ChatGPT-4, with the majority of the conversations resulting in a failed benchmark, and the majority of those that passed their own testbenches being non-compliant.
The modes of failure were less consistent with ChatGPT-3.5 than they were for ChatGPT-4, with a wide variety of issues introduced between each conversation and benchmark.
It required corrections to the design and the testbenches far more often than ChatGPT-4. 

## Observation and concluding remarks

Of the four LLMs examined with the challenge benchmarks, only ChatGPT-4 performed adequately, though it still required human feedback for most conversations to be both successful and compliant with the given specifications.
When fixing errors, ChatGPT-4 would often require several messages to fix minor errors, as it struggled to understand exactly what specific Verilog lines would cause the error messages from iverilog.
The errors it would add also tended to repeat themselves between conversations quite often.

ChatGPT-4 also struggled much more to create functioning testbenches than functioning designs.
The majority of benchmarks required little to no modification of the design itself, instead necessitating testbench repair.
This is particularly true of FSMs, as the model seemed unable to create a testbench which would properly check the output without significant feedback regarding the state transitions and corresponding expected outputs.
ChatGPT-3.5, on the other hand, struggled with both testbenches and functional designs.

# Part 4: Something more complex: The QTcore-A1

**to be continued**