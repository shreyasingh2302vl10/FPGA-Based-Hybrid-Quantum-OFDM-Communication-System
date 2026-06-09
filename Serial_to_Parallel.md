# LOGIC FLOW
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/5cac408fd44f8d4a473fe50e0d6765b26eadfad7/Serial2Parallel.png)
# CODE
```verilog
`timescale 1ns / 1ps

module serial_to_parallel #(
    parameter NUM_SUBCARRIERS = 4,
    parameter BIT_WIDTH = 2
)(
    input  wire clk,
    input  wire rst_n,
    input  wire [1:0] in_I,
    input  wire [1:0] in_Q,
    input  wire valid_in,
    output reg [(NUM_SUBCARRIERS*BIT_WIDTH)-1:0] parallel_I,
    output reg [(NUM_SUBCARRIERS*BIT_WIDTH)-1:0] parallel_Q,
    output reg parallel_valid
);

    reg [1:0] bit_counter; 

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            bit_counter    <= 2'd0;
            parallel_valid <= 1'b0;
            parallel_I     <= 0;
            parallel_Q     <= 0;
        end else if (valid_in) begin
            // Shift 2-bit symbols into the parallel bus
            parallel_I <= {parallel_I[(NUM_SUBCARRIERS*BIT_WIDTH)-3:0], in_I};
            parallel_Q <= {parallel_Q[(NUM_SUBCARRIERS*BIT_WIDTH)-3:0], in_Q};
            
            // Counter loop (0 to 3)
            if (bit_counter == 2'd3) begin
                bit_counter <= 2'd0;
            end else begin
                bit_counter <= bit_counter + 1'b1;
            end
            
            // Lookahead flag: asserts high on the exact cycle the data registers stabilize
            parallel_valid <= (bit_counter == 2'd3);
        end else begin
            parallel_valid <= 1'b0; 
        end
    end
endmodule
```
