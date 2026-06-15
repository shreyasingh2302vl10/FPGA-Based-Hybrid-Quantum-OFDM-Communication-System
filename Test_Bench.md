# INPUT
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/59a8545b92e8182d7d2607cb4d5287fd1d877e9c/Screenshot%202026-06-12%20032209.png
)
### Transmitter I/Q Symbols:
    I  Q
    01 11
    11 11
    11 11
    11 01
    11 11
    01 11
    11 01
    11 01
# OUTPUT
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/59a8545b92e8182d7d2607cb4d5287fd1d877e9c/Screenshot%202026-06-12%20032846.png)
### Recieved Output
    ff80ff00
    005bffa5
    ff00ff80
    005b015b
    ff800000
    ffa5005b
    00000180
    ffa500a5
# CODE 
```verilog
`timescale 1ns / 1ps

module tb_transmitter;
    // Clock and Reset
    reg clk;
    reg rst_n;
    
    // Inputs to DUT
    reg data_in;
    reg valid_in;
    
    // Outputs from DUT (Matches your transmitter_top)
    wire [1:0] qpsk_I_out;
    wire [1:0] qpsk_Q_out;
    wire [31:0] ifft_out_tdata; // IP core 32-bit output de raha hai
    wire ifft_out_tvalid;

    // Instantiate UUT
    transmitter_top uut (
        .clk(clk), 
        .rst_n(rst_n), 
        .data_in(data_in), 
        .valid_in(valid_in),
        .qpsk_I_out(qpsk_I_out),
        .qpsk_Q_out(qpsk_Q_out),
        .ifft_out_tdata(ifft_out_tdata),
        .ifft_out_tvalid(ifft_out_tvalid)
    );

    // Clock generation (100MHz)
    always #5 clk = ~clk;

    // Task for sending a bit
    task send_bit(input reg val);
        begin
            @(posedge clk);
            data_in <= val;
            valid_in <= 1;
            @(posedge clk); 
            valid_in <= 0; // Valid ko ek pulse ki tarah bhejein
        end
    endtask

    initial begin
        // Initialize
        clk = 0; rst_n = 0; data_in = 0; valid_in = 0;
        
        // Reset sequence (Reset ko 20ns tak rakhein)
        #20 rst_n = 1;
        
        // 16 bits ka data bhejein (ya jo bhi aapka IP core expect kar raha hai)
        repeat (16) begin
            send_bit($urandom_range(0, 1));
        end

        // Wait for IFFT processing latency
        #2000;
        
        $display("Simulation Complete!");
        $finish;
    end
endmodule
```
