# INPUT
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/39d9ce4e0652a90e04abe3ea82ab185ccafc4d52/Transmitter.png
)
### Transmitter I/Q Symbols:
        I  Q
       11 11
       01 01
       11 11
       11 01
       11 11
       11 01
       01 01
       01 11
# OUTPUT
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/39d9ce4e0652a90e04abe3ea82ab185ccafc4d52/OUTPUT.png)
### Recieved Output:
       0000ff80
       ff2500db
       ff80ff00
       00dbfe6f
       ff00ff80
       ffdb0025
       ff800000
       00250091
# CODE 
```verilog
`timescale 1ns / 1ps

module tb_ofdm_top();

   
    reg clk;
    reg rst_n;
    
    // Inputs to the OFDM System
    reg data_in;
    reg valid_in;

    // Outputs from the OFDM System
    wire [31:0] final_fft_data;
    wire        final_fft_valid;

    // 1 Symbol = 2 bits (QPSK style)
    // Total bits for 8 symbols = 8 * 2 = 16 bits
    parameter NUM_SYMBOLS     = 8;
    parameter BITS_PER_SYMBOL = 2;
    
    integer total_bits = NUM_SYMBOLS * BITS_PER_SYMBOL; // Evaluates to 16
    integer i;

    // Instantiate the Top Module (Device Under Test)
    ofdm_top uut (
        .clk(clk),
        .rst_n(rst_n),
        .data_in(data_in),
        .valid_in(valid_in),
        .final_fft_data(final_fft_data),
        .final_fft_valid(final_fft_valid)
    );

    // 100MHz Clock Generation
    always #5 clk = ~clk;

    // Test Sequence
    initial begin
        // Signal Initialization
        clk = 0;
        rst_n = 0;
        data_in = 0;
        valid_in = 0;
        
        // Reset sequence
        #50;
        @(posedge clk);
        rst_n <= 1;   
        
        // Wait for system stabilization
        repeat(5) @(posedge clk);
        
        $display("Starting Simulation: Sending %0d symbols (%0d bits total)...", NUM_SYMBOLS, total_bits);
        
        // 1. Turn valid_in ON once at the start
        @(posedge clk);
        valid_in <= 1;
        
        // 2. Loop to stream exactly 16 bits (8 symbols)
        for (i = 0; i < total_bits; i = i + 1) begin
            data_in <= $random;
            @(posedge clk);
        end
        
        // 3. Turn valid_in and data_in OFF exactly after the 16th bit
        valid_in <= 0;
        data_in  <= 0;
        $display(">> Sent 8 symbols (16 bits). valid_in turned OFF. <<");
        
        // Wait for the pipeline processing latency to completely finish
        $display("Waiting for FFT pipeline to flush completely...");
        #5000;  
        
        $display("Simulation Finished.");
        $finish;
    end

    // Signal Monitor for Console Debugging
    initial begin
        $monitor("Time=%0t | Valid_in=%b | Data_in=%b | FFT_Valid=%b | FFT_Data=%h", 
                 $time, valid_in, data_in, final_fft_valid, final_fft_data);
    end

endmodule
```
