```verilog
`timescale 1ns / 1ps

module final_decision_demapper(
    input  wire clk,
    input  wire rst_n,
    input  wire [31:0] fft_data_in,     // Raw input vector straight from xfft_1
    input  wire        fft_valid_in,    // Valid handshake from xfft_1
    output reg  [1:0]  parallel_bits,   // Decoded QPSK phase bit pairs
    output reg         bit_valid
);

    wire signed [15:0] I_chan = fft_data_in[15:0];
    wire signed [15:0] Q_chan = fft_data_in[31:16];

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            parallel_bits <= 2'b00;
            bit_valid     <= 1'b0;
        end else begin
            if (fft_valid_in) begin
                bit_valid <= 1'b1;
                
               
                parallel_bits <= { (Q_chan >= 16'sd0) ? 1'b1 : 1'b0, 
                                   (I_chan >= 16'sd0) ? 1'b1 : 1'b0 };
                                   
            end else begin
                bit_valid <= 1'b0;
                
            end
        end
    end

endmodule
```
