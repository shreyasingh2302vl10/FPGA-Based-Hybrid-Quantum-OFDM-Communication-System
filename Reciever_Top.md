```verilog
`timescale 1ns / 1ps

module reciever_top(
    input  wire clk,
    input  wire rst_n,
    input  wire [31:0] rx_data_in,       // Direct IFFT data input from Transmitter
    input  wire        rx_valid_in,      // Direct IFFT valid input
    output wire [31:0] fft_out_tdata,
    output wire        fft_out_tvalid,
   
    output wire [1:0]  final_rx_bits,
    output wire        final_rx_valid
);

    // FFT Frame tracking counter (For 8-point FFT)
    reg [2:0] sample_counter;
    wire      internal_tlast; 

    // --- Dynamic TLAST Generation Logic ---
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            sample_counter <= 3'd0;
        end else begin
            if (rx_valid_in) begin
                if (sample_counter == 3'd7) begin
                    sample_counter <= 3'd0; // Reset for the next symbol frame
                end else begin
                    sample_counter <= sample_counter + 1'b1;
                end
            end
        end
    end

 
    wire [15:0] w_config_tdata = 16'h0007;

    // --- Xilinx FFT Block Instantiation ---
    xfft_1 u_fft_receiver (
        .aclk(clk),
        
        // Configuration Channel (Set to 8-Point Forward FFT)
        .s_axis_config_tdata(w_config_tdata),   
        .s_axis_config_tvalid(1'b1),
        
        // Slave Data Input Channel (Connected directly to Receiver Inputs)
        .s_axis_data_tdata(rx_data_in),   
        .s_axis_data_tvalid(rx_valid_in), 
        .s_axis_data_tready(),            // Open loop (No backpressure)
        .s_axis_data_tlast(internal_tlast),
        
        // Master Data Output Channel (Decoded Frequency Spectrum)
        .m_axis_data_tdata(fft_out_tdata),
        .m_axis_data_tvalid(fft_out_tvalid),
        .m_axis_data_tready(1'b1)         // Always ready to accept output
    );

    // --- Demapper Instance ---
    final_decision_demapper u_demapper (
        .clk(clk),
        .rst_n(rst_n),
        .fft_data_in(fft_out_tdata),
        .fft_valid_in(fft_out_tvalid),
        .parallel_bits(final_rx_bits),   // Mapped straight to output ports
        .bit_valid(final_rx_valid)       // Mapped straight to output ports
    );

endmodule
```
