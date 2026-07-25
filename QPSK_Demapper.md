```verilog
`timescale 1ns / 1ps

module qpsk_demapper (
    input  wire        clk,
    input  wire        rst_n,
    
    // --- Input Interface from Receiver FFT ---
    input  wire [15:0] fft_tdata_in,   // [15:8] = Q channel, [7:0] = I channel
    input  wire        fft_tvalid_in,  // Valid signal from FFT
    
    // --- Outputs ---
    output reg  [7:0]  mapped_I_out,   // +8 or -8 constellation level for I
    output reg  [7:0]  mapped_Q_out,   // +8 or -8 constellation level for Q
    output reg  [1:0]  demap_2bit_out, // [1] = I bit, [0] = Q bit
    output reg         demap_valid     // Valid indicator for demapped output
);

    // Wire extraction for signed arithmetic
    wire signed [7:0] rx_I_signed;
    wire signed [7:0] rx_Q_signed;

    // Direct Extraction from 16-bit FFT output bus
    assign rx_I_signed = $signed(fft_tdata_in[7:0]);
    assign rx_Q_signed = $signed(fft_tdata_in[15:8]);

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            mapped_I_out   <= 8'sd0;
            mapped_Q_out   <= 8'sd0;
            demap_2bit_out <= 2'b00;
            demap_valid    <= 1'b0;
        end else begin
            // Pass valid through
            demap_valid <= fft_tvalid_in;

            if (fft_tvalid_in) begin
                // --- Demapping Logic for I Channel ---
                if (rx_I_signed >= 0) begin
                    mapped_I_out      <= 8'sd8;   // +8 Level
                    demap_2bit_out[1] <= 1'b1;     // Decoded Bit 1
                end else begin
                    mapped_I_out      <= -8'sd8;  // -8 Level
                    demap_2bit_out[1] <= 1'b0;     // Decoded Bit 0
                end

                // --- Demapping Logic for Q Channel ---
                if (rx_Q_signed >= 0) begin
                    mapped_Q_out      <= 8'sd8;   // +8 Level
                    demap_2bit_out[0] <= 1'b1;     // Decoded Bit 1
                end else begin
                    mapped_Q_out      <= -8'sd8;  // -8 Level
                    demap_2bit_out[0] <= 1'b0;     // Decoded Bit 0
                end
            end else begin
                mapped_I_out   <= 8'sd0;
                mapped_Q_out   <= 8'sd0;
                demap_2bit_out <= 2'b00;
            end
        end
    end

endmodule
```
