```verilog
`timescale 1ns / 1ps

module ifft_top (
    input clk,
    input rst,
    input signed [15:0] X0_r, input signed [15:0] X0_i,
    input signed [15:0] X1_r, input signed [15:0] X1_i,
    input signed [15:0] X2_r, input signed [15:0] X2_i,
    input signed [15:0] X3_r, input signed [15:0] X3_i,
    output signed [15:0] x0_r, output signed [15:0] x0_i,
    output signed [15:0] x1_r, output signed [15:0] x1_i,
    output signed [15:0] x2_r, output signed [15:0] x2_i,
    output signed [15:0] x3_r, output signed [15:0] x3_i
);

    localparam signed [15:0] W4_0_R = 16'h7FFF;
    localparam signed [15:0] W4_0_I = 16'h0000;
    localparam signed [15:0] W4_1_R = 16'h0000;
    localparam signed [15:0] W4_1_I = 16'h7FFF;

    wire signed [15:0] s1_out0_r, s1_out0_i;
    wire signed [15:0] s1_out1_r, s1_out1_i;
    wire signed [15:0] s1_out2_r, s1_out2_i;
    wire signed [15:0] s1_out3_r, s1_out3_i;

    butterfly_element_ifft stage1_bf1 (
        .clk(clk), .rst(rst),
        .in_a_r(X0_r), .in_a_i(X0_i),
        .in_b_r(X2_r), .in_b_i(X2_i),
        .twiddle_r(W4_0_R), .twiddle_i(W4_0_I),
        .out_a_r(s1_out0_r), .out_a_i(s1_out0_i),
        .out_b_r(s1_out2_r), .out_b_i(s1_out2_i)
    );

    butterfly_element_ifft stage1_bf2 (
        .clk(clk), .rst(rst),
        .in_a_r(X1_r), .in_a_i(X1_i),
        .in_b_r(X3_r), .in_b_i(X3_i),
        .twiddle_r(W4_1_R), .twiddle_i(W4_1_I),
        .out_a_r(s1_out1_r), .out_a_i(s1_out1_i),
        .out_b_r(s1_out3_r), .out_b_i(s1_out3_i)
    );

    butterfly_element_ifft stage2_bf1 (
        .clk(clk), .rst(rst),
        .in_a_r(s1_out0_r), .in_a_i(s1_out0_i),
        .in_b_r(s1_out1_r), .in_b_i(s1_out1_i),
        .twiddle_r(W4_0_R), .twiddle_i(W4_0_I),
        .out_a_r(x0_r), .out_a_i(x0_i),
        .out_b_r(x2_r), .out_b_i(x2_i)
    );

    butterfly_element_ifft stage2_bf2 (
        .clk(clk), .rst(rst),
        .in_a_r(s1_out2_r), .in_a_i(s1_out2_i),
        .in_b_r(s1_out3_r), .in_b_i(s1_out3_i),
        .twiddle_r(W4_0_R), .twiddle_i(W4_0_I),
        .out_a_r(x1_r), .out_a_i(x1_i),
        .out_b_r(x3_r), .out_b_i(x3_i)
    );

endmodule
```
