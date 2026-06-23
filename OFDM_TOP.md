```verilog
`timescale 1ns / 1ps

module ofdm_top (
    input  wire clk,
    input  wire rst_n,
    input  wire data_in,      // Original bit stream
    input  wire valid_in,
    
    // Debug outputs
    output wire [31:0] final_fft_data,
    output wire        final_fft_valid
);

    // Transmitter aur Receiver ke beech ka connection
    wire [31:0] w_ifft_to_rx;
    wire        w_ifft_valid_to_rx;

    // 1. Transmitter Instance
    transmitter_top u_tx (
        .clk(clk),
        .rst_n(rst_n),
        .data_in(data_in),
        .valid_in(valid_in),
        .qpsk_I_out(), 
        .qpsk_Q_out(),
        .ifft_out_tdata(w_ifft_to_rx),
        .ifft_out_tvalid(w_ifft_valid_to_rx)
    );

    // 2. Receiver Instance
    reciever_top u_rx (
        .clk(clk),
        .rst_n(rst_n),
        .rx_data_in(w_ifft_to_rx),
        .rx_valid_in(w_ifft_valid_to_rx),
        .fft_out_tdata(final_fft_data),
        .fft_out_tvalid(final_fft_valid)
    );

endmodule
```
