```verilog
`timescale 1ns / 1ps

module add_cyclic_prefix (
    input clk,
    input rst,
    input signed [15:0] x0_r, input signed [15:0] x0_i,
    input signed [15:0] x1_r, input signed [15:0] x1_i,
    input signed [15:0] x2_r, input signed [15:0] x2_i,
    input signed [15:0] x3_r, input signed [15:0] x3_i,
    output reg signed [15:0] cp_out0_r, output reg signed [15:0] cp_out0_i,
    output reg signed [15:0] cp_out1_r, output reg signed [15:0] cp_out1_i,
    output reg signed [15:0] cp_out2_r, output reg signed [15:0] cp_out2_i,
    output reg signed [15:0] cp_out3_r, output reg signed [15:0] cp_out3_i,
    output reg signed [15:0] cp_out4_r, output reg signed [15:0] cp_out4_i
);

    always @(posedge clk or posedge rst) begin
        if (rst) begin
            cp_out0_r <= 16'h0000; cp_out0_i <= 16'h0000;
            cp_out1_r <= 16'h0000; cp_out1_i <= 16'h0000;
            cp_out2_r <= 16'h0000; cp_out2_i <= 16'h0000;
            cp_out3_r <= 16'h0000; cp_out3_i <= 16'h0000;
            cp_out4_r <= 16'h0000; cp_out4_i <= 16'h0000;
        end else begin
            cp_out0_r <= x3_r;
            cp_out0_i <= x3_i;
            
            cp_out1_r <= x0_r;
            cp_out1_i <= x0_i;
            
            cp_out2_r <= x1_r;
            cp_out2_i <= x1_i;
            
            cp_out3_r <= x2_r;
            cp_out3_i <= x2_i;
            
            cp_out4_r <= x3_r;
            cp_out4_i <= x3_i;
        end
    end

endmodule
```
