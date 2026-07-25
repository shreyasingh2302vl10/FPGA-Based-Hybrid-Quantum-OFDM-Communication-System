# LOGIC FLOW
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/bf25332cde6eb83c3b0475e1368dca2afb388cca/QPSK_Mapper.png)


# CODE
```verilog
`timescale 1ns / 1ps

module qpsk_mapper (
    input  wire       clk,
    input  wire       rst_n,
    input  wire       data_in,
    input  wire       valid_in,
    output reg [7:0]  qpsk_i,     // 8-bit output
    output reg [7:0]  qpsk_q,     // 8-bit output
    output reg        valid_out
);

    reg bit_phase;
    reg buffer_bit;

    // Direct 8-bit signed decimal representation
    localparam [7:0] POSITIVE_ONE = 8'sd1;   //  1
    localparam [7:0] NEGATIVE_ONE = -8'sd1;  // -1 (hardware me 8'b11111111 banega)

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            bit_phase  <= 1'b0;
            buffer_bit <= 1'b0;
            qpsk_i     <= 8'd0;
            qpsk_q     <= 8'd0;
            valid_out  <= 1'b0;
        end else begin
            if (valid_in) begin
                if (bit_phase == 1'b0) begin
                    buffer_bit <= data_in;
                    bit_phase  <= 1'b1;
                    valid_out  <= 1'b0;
                end else begin
                    bit_phase <= 1'b0;
                    valid_out <= 1'b1;
                    
                    // Bit mapping: 0 -> -1, 1 -> +1
                    qpsk_i <= (buffer_bit == 1'b0) ? NEGATIVE_ONE : POSITIVE_ONE;
                    qpsk_q <= (data_in    == 1'b0) ? NEGATIVE_ONE : POSITIVE_ONE;
                end
            end else begin
                valid_out <= 1'b0;
            end
        end
    end
endmodule
```
