# LOGIC FLOW
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/1d19166594b56394acb60a804cce05691701376d/Butterfly_ifft.png)
# CODE
```verilog 
`timescale 1ns / 1ps

module butterfly_element_ifft (
    input clk,
    input rst,
    input signed [15:0] in_a_r,
    input signed [15:0] in_a_i,
    input signed [15:0] in_b_r,
    input signed [15:0] in_b_i,
    input signed [15:0] twiddle_r,
    input signed [15:0] twiddle_i,
    output reg signed [15:0] out_a_r,
    output reg signed [15:0] out_a_i,
    output reg signed [15:0] out_b_r,
    output reg signed [15:0] out_b_i
);

    wire signed [16:0] add_r = in_a_r + in_b_r;
    wire signed [16:0] add_i = in_a_i + in_b_i;
    wire signed [16:0] sub_r = in_a_r - in_b_r;
    wire signed [16:0] sub_i = in_a_i - in_b_i;

    reg signed [32:0] prod_r;
    reg signed [32:0] prod_i;

    always @(posedge clk or posedge rst) begin
        if (rst) begin
            out_a_r <= 16'h0000;
            out_a_i <= 16'h0000;
            out_b_r <= 16'h0000;
            out_b_i <= 16'h0000;
        end else begin
            out_a_r <= add_r[16:1];
            out_a_i <= add_i[16:1];

            prod_r <= (sub_r * twiddle_r) - (sub_i * twiddle_i);
            prod_i <= (sub_r * twiddle_i) + (sub_i * twiddle_r);

            out_b_r <= prod_r[30:15];
            out_b_i <= prod_i[30:15];
        end
    end
endmodule
```
