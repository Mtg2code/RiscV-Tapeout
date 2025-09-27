# My RISC-V Project Journal -  Day 5: RTL Optimization 

Welcome to Day 5 of the RTL workshop\! Today, we learned how to write Verilog that is not only functional but also **optimized** and **scalable** for hardware synthesis. The focus was on making code structures like `if-else` statements, `for` loops, and `generate` blocks work correctly and efficiently in silicon, especially by avoiding **inferred latches**.

-----

## 1\. Conditional Logic: IF and CASE Constructs

### IF-ELSE Statements

The `if` statement is a conditional statement that uses Boolean conditions to determine which blocks of Verilog code to execute.

  * **Hardware Implementation**: Translates into a **Multiplexer** (MUX).
  * **Usage**: Used for **Priority Logic** and must be inside an `always` block. The variable being assigned should be declared as a `reg`.

#### Syntax for IF Statement

```verilog
if <cond> begin
    .....
    .....
end else begin
    .....
    .....
end
```

#### Syntax for IF- ELSE-IF Statement

```verilog
if <cond1> begin
    .....
    // executes cb1
    .....
end else if <cond2> begin
    .....
    // executes cb2
    .....
end else if <cond3> begin
    .....
    // executes cb3
    .....
end else begin
    .....
    // executes cb4
    .....
end
```

#### Cautions with using IF Statements

**Inferred latches** serve as a warning sign. They represent a bad coding style resulting from incomplete `if` statements (e.g., a missing `else`). When the hardware isn't informed on the decision, it will latch (retain the previous value). This must be avoided unless explicitly intended (e.g., in certain counter designs).

-----

### CASE Statements

  * **Hardware Implementation**: Translates into a **Multiplexer** (MUX).
  * **Usage**: Used inside an `always` block, and the assigned variable should be a `reg` variable.

#### Basic Case Structure

```verilog
reg y;
always @ (*) begin
    case (sel)
        2'b00: begin
              ....
              end
        2'b01: begin
              ....
              end
              .
              .
              .
    endcase
end
```

#### Caveats in CASE Statements

1.  **Incomplete Case Structure**: An incomplete structure may lead to inferred latches.

      * **Solution**: Always code `case` statements with a `default` condition to cover all possibilities.

    <!-- end list -->

    ```verilog
    reg y;
    always @ (*) begin
        case (sel)
            2'b00: begin
                  ....
                  end
            2'b01: begin
                  ....
                  end
                  .
                  .
        default: begin
                 ....
                 end
        endcase
    end
    ```

2.  **Partial Assignments**: Not specifying the values for **all** output signals in all segments of the case statement will also create inferred latches.

      * **Solution**: Assign all the output signals in all the segments of the case statement.

-----

## 2\. Inferred Latches in Verilog ⚠️

**Inferred latches** occur when a combinational logic block (`always @(*)`) does not assign a value to a variable in all possible execution paths.

### Example of Latch Inference

| Verilog Code (Problematic) | Problem |
| :--- | :--- |
| ` verilog module ex ( input wire a, b, sel, output reg y ); always @(a, b, sel) begin if (sel == 1'b1) y = a; // No 'else' - y is not assigned when sel == 0 end endmodule  ` | **Problem**: When `sel` is 0, `y` is not assigned, so a latch is inferred. |

### Solution: Complete Assignments

To ensure pure combinational logic (no latches), every output must be assigned in every path.

```verilog
module ex (
    input wire a, b, sel,
    output reg y
);
    always @(a, b, sel) begin
        case (sel)
            1'b1 : y = a;
            default : y = 1'b0; // Default assignment prevents the latch
        endcase
    end
endmodule
```

-----

## 3\. Labs on Conditional Synthesis (Latches vs. Gates) 🔬

These labs demonstrate how synthesis translates conditional logic and the pitfalls of incomplete assignments.

### Lab 1: Incomplete If Statement

| Code File | Synthesis Analysis |
| :--- | :--- |
| ` verilog module incomp_if (input i0, input i1, input i2, output reg y); always @(*) begin if (i0) y <= i1; end endmodule  ` | The synthesized design has a **D Latch inferred** due to incomplete `if` structure (missing `else` statement). |

-----

### Lab 2: Synthesis Result of Lab 1

| Code File | Synthesis Analysis |
| :--- | :--- |
| **Verilog File** (Reference Lab 1 code) | **Realization of Logic** shows the inferred latch. |

-----

### Lab 3: Nested If-Else

| Code File | Synthesis Analysis |
| :--- | :--- |
| ` verilog module incomp_if2 (input i0, input i1, input i2, input i3, output reg y); always @(*) begin if (i0) y <= i1; else if (i2) y <= i3; end endmodule  ` | **Inferred Latches** are present because `y` is unassigned when both `i0` and `i2` are low. |

-----

### Lab 4: Synthesis Result of Lab 3

| Code File | Synthesis Analysis |
| :--- | :--- |
| **Verilog File** (Reference Lab 3 code) | **Realization of Logic** shows the inferred latch. |

-----

### Lab 5: Complete Case Statement

| Code File | Synthesis Analysis |
| :--- | :--- |
| ` verilog module comp_case (input i0, input i1, input i2, input [1:0] sel, output reg y); always @(*) begin case(sel) 2'b00 : y = i0; 2'b01 : y = i1; default : y = i2; endcase end endmodule  ` | The logic is a proper **combinational circuit** (MUX) because the `default` sets the output for all cases. |

-----

### Lab 6: Synthesis Result of Lab 5

| Code File | Synthesis Analysis |
| :--- | :--- |
| **Verilog File** (Reference Lab 5 code) | **Realization of Logic** confirms the clean combinational MUX. |

-----

### Lab 7: Incomplete Case Handling

| Code File | Synthesis Analysis |
| :--- | :--- |
| ` verilog module bad_case ( input i0, input i1, input i2, input i3, input [1:0] sel, output reg y ); always @(*) begin case(sel) 2'b00: y = i0; 2'b01: y = i1; 2'b10: y = i2; 2'b1?: y = i3; // '?' is a wildcard; be careful with incomplete cases! endcase end endmodule  ` | **Risk of Latch.** The synthesized design has a **D Latch inferred** due to incomplete case structure. |

-----

### Lab 8: Partial Assignments in Case

| Code File | Synthesis Analysis |
| :--- | :--- |
| ` verilog module partial_case_assign ( input i0, input i1, input i2, input [1:0] sel, output reg y, output reg x ); always @(*) begin case(sel) 2'b00: begin y = i0; x = i2; end 2'b01: y = i1; // Latch inferred for x here default: begin x = i1; y = i2; end endcase end endmodule  ` | **Latch Inferred for `x`**: Output `x` is unassigned in the `2'b01` branch, creating a latch on `x` only. |

> **Note:** Steps to perform the above labs are shown in [Day 1](https://github.com/Ahtesham18112011/RTL_workshop/tree/main/Day_1).

-----

## 4\. For Loops in Verilog

A **`for` loop** is used within procedural blocks to execute statements multiple times.

### Syntax and Synthesizability

| Type | Placement | Primary Use |
| :--- | :--- | :--- |
| **FOR Statement** | Used **inside** the `always` block. | Used for evaluating expressions (Behavioral Replication). |

```verilog
for (initialization; condition; increment) begin
    // Statements to execute
end
```

  - **Synthesizable only if the number of iterations is fixed at compile time**.

#### Example: 4-to-1 MUX Using a For Loop

```verilog
module mux_4to1_for_loop (
    input wire [3:0] data, // 4 input lines
    input wire [1:0] sel,  // 2-bit select
    output reg y           // Output
);
    integer i;
    always @(data, sel) begin
        y = 1'b0; // Default output
        for (i = 0; i < 4; i = i + 1) begin
            if (i == sel)
                y = data[i];
        end
    end
endmodule
```

-----

## 5\. Generate Blocks in Verilog

A **`generate` block** is used to create hardware structures like module instances at compile time.

| Type | Placement | Primary Use |
| :--- | :--- | :--- |
| **GENERATE Statement** | Used **outside** the `always` block. | Used for instantiating/replicating Hardwares (Structural Replication). |

#### Example

```verilog
genvar i;
generate
    for (i = 0; i < 4; i = i + 1) begin : gen_loop
        and_gate and_inst (.a(in[i]), .b(in[i+1]), .y(out[i]));
    end
endgenerate
```

-----

## 6\. What is an RCA (Ripple Carry Adder)?

An **RCA (Ripple Carry Adder)** adds binary numbers using a chain of full adders.

  * To add $n$ bits, you need $n$ full adders.
  * Each carry-out connects to the carry-in of the next stage, causing the carry to **ripple** along the chain, which introduces significant delay.

-----

## 7\. Labs on Loops and Generate Blocks (Scalable Design) ⚙️

These labs demonstrate writing scalable and structurally repetitive code.

### Lab 9: 4-to-1 MUX Using For Loop

| Code File | Hardware Purpose |
| :--- | :--- |
| ` verilog module mux_generate ( input i0, input i1, input i2, input i3, input [1:0] sel, output reg y ); wire [3:0] i_int; assign i_int = {i3, i2, i1, i0}; integer k; always @(*) begin y = 1'b0; for (k = 0; k < 4; k = k + 1) begin if (k == sel) y = i_int[k]; end end endmodule  ` | **Efficient, Scalable MUX** (Behavioral Replication). |

-----

### Lab 10: 8-to-1 Demux Using Case

| Code File | Hardware Purpose |
| :--- | :--- |
| ` verilog module demux_case ( output o0, output o1, output o2, output o3, output o4, output o5, output o6, output o7, input [2:0] sel, input i ); reg [7:0] y_int; assign {o7, o6, o5, o4, o3, o2, o1, o0} = y_int; always @(*) begin y_int = 8'b0; case(sel) 3'b000 : y_int[0] = i; 3'b001 : y_int[1] = i; 3'b010 : y_int[2] = i; 3'b011 : y_int[3] = i; 3'b100 : y_int[4] = i; 3'b101 : y_int[5] = i; 3'b110 : y_int[6] = i; 3'b111 : y_int[7] = i; endcase end endmodule  ` | **8-to-1 Demux** (Static, non-scalable code). |

-----

### Lab 11: 8-to-1 Demux Using For Loop

| Code File | Hardware Purpose |
| :--- | :--- |
| ` verilog module demux_generate ( output o0, output o1, output o2, output o3, output o4, output o5, output o6, output o7, input [2:0] sel, input i ); reg [7:0] y_int; assign {o7, o6, o5, o4, o3, o2, o1, o0} = y_int; integer k; always @(*) begin y_int = 8'b0; for (k = 0; k < 8; k = k + 1) begin if (k == sel) y_int[k] = i; end end endmodule  ` | **Scalable 8-to-1 Demux** (Behavioral Replication). |

-----

### Lab 12: 8-bit Ripple Carry Adder with Generate Block

| Code File | Hardware Purpose |
| :--- | :--- |
| ` verilog module rca ( input [7:0] num1, input [7:0] num2, output [8:0] sum ); wire [7:0] int_sum; wire [7:0] int_co; genvar i; generate for (i = 1; i < 8; i = i + 1) begin fa u_fa_1 (.a(num1[i]), .b(num2[i]), .c(int_co[i-1]), .co(int_co[i]), .sum(int_sum[i])); end endgenerate fa u_fa_0 (.a(num1[0]), .b(num2[0]), .c(1'b0), .co(int_co[0]), .sum(int_sum[0])); assign sum[7:0] = int_sum; assign sum[8] = int_co[7]; endmodule  ` | **Structural, Parameterized 8-bit RCA** (Structural Replication). |

**Full Adder Module:**

```verilog
module fa (input a, input b, input c, output co, output sum);
    assign {co, sum} = a + b + c;
endmodule
```

-----

## Summary

  * **Latch Avoidance:** Always use **complete** `if-else` or `case` statements (e.g., include `else` or `default`) in **`always @(*)`** blocks.
  * **Scalability:** **`For` loops** and **`generate` blocks** are powerful tools for writing code that easily scales to different bit-widths and sizes.
  * **Clean Code:** Always ensure every signal is assigned in every possible execution path for combinational logic.

-----