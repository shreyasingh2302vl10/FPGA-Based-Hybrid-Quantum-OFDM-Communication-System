```verilog
`timescale 1ns / 1ps

module tb_transmitter;
    reg clk;
    reg rst_n;
    reg data_in;
    reg valid_in;
    
    // Inhe wire ki tarah define karna zaroori hai
    wire [1:0] qpsk_I_out;
    wire [1:0] qpsk_Q_out;
    wire [31:0] ifft_out_tdata;
    wire ifft_out_tvalid;

    // UUT Instance
    transmitter_top uut (
        .clk(clk), .rst_n(rst_n), .data_in(data_in), .valid_in(valid_in),
        .qpsk_I_out(qpsk_I_out),
        .qpsk_Q_out(qpsk_Q_out),
        .ifft_out_tdata(ifft_out_tdata),
        .ifft_out_tvalid(ifft_out_tvalid)
    );

    always #5 clk = ~clk; // 100MHz clock

    initial begin
        // Init
        clk = 0; rst_n = 0; data_in = 0; valid_in = 0;
        #20 rst_n = 1; 

        // Data pattern bhejo (Random nahi, fixed sequence bhejo taaki verify kar sako)
        // Ye 16 bits ka sequence hai (8 symbols banenge)
        repeat (16) begin
            @(posedge clk);
            valid_in = 1;
            data_in = $random; // Har clock par data change hoga
        end
        
        @(posedge clk);
        valid_in = 0;
        
        #2000 $finish; 
    end
endmodule
```
