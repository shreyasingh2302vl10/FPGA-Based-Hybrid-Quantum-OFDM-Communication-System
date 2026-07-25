# DATA TRANSMISSION
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/4fa94a16284469a0bf4263295cdfdcab3678282d/Complete_Transmission_Block.jpeg)
# CODE 
```verilog
`timescale 1ns / 1ps

module tb_ofdm_top;

    // -------------------------------------------------------------
    // Testbench Clock & Reset Signals
    // -------------------------------------------------------------
    reg        clk;
    reg        rst_n;

    // -------------------------------------------------------------
    // Transmitter Inputs
    // -------------------------------------------------------------
    reg        tx_data_in;
    reg        tx_valid_in;

    // -------------------------------------------------------------
    // Top Module Outputs
    // -------------------------------------------------------------
    wire [15:0] tx_ifft_out_tdata;
    wire        tx_ifft_out_tvalid;
    wire        tx_ifft_out_tlast;

    wire [15:0] rx_fft_out_tdata;
    wire        rx_fft_out_tvalid;
    wire        rx_fft_out_tlast;

    wire [7:0]  rx_fft_I_out;
    wire [7:0]  rx_fft_Q_out;

    // -------------------------------------------------------------
    // Unit Under Test (UUT) Instance
    // -------------------------------------------------------------
    ofdm_top uut (
        .clk                 (clk),
        .rst_n               (rst_n),
        
        // Tx Inputs
        .tx_data_in          (tx_data_in),
        .tx_valid_in         (tx_valid_in),
        
        // Tx IFFT Outputs
        .tx_ifft_out_tdata   (tx_ifft_out_tdata),
        .tx_ifft_out_tvalid  (tx_ifft_out_tvalid),
        .tx_ifft_out_tlast   (tx_ifft_out_tlast),
        
        // Rx FFT Outputs
        .rx_fft_out_tdata    (rx_fft_out_tdata),
        .rx_fft_out_tvalid   (rx_fft_out_tvalid),
        .rx_fft_out_tlast    (rx_fft_out_tlast),
        
        // Parsed I/Q Outputs from Receiver
        .rx_fft_I_out        (rx_fft_I_out),
        .rx_fft_Q_out        (rx_fft_Q_out)
    );

    // -------------------------------------------------------------
    // 100 MHz Clock (10ns Period)
    // -------------------------------------------------------------
    always #5 clk = ~clk;

    integer i;
    // Test Stream: Exact 16 bits = 8 QPSK Symbols (1 Frame for 8-point IFFT/FFT)
    reg [15:0] test_bits = 16'b10_00_10_11_11_11_11_11;

    // -------------------------------------------------------------
    // Stimulus Generation
    // -------------------------------------------------------------
    initial begin
        clk         = 0;
        rst_n       = 0;
        tx_data_in  = 0;
        tx_valid_in = 0;

        // Reset Pulse
        #50;
        rst_n = 1;
        #20;

        $display("\n=======================================================");
        $display("   STARTING TOP OFDM (TX IFFT + RX FFT) SIMULATION ");
        $display("=======================================================\n");

        // Drive 16 bits serially on posedge clk
        for (i = 15; i >= 0; i = i - 1) begin
            @(posedge clk);
            tx_data_in  <= test_bits[i];
            tx_valid_in <= 1'b1;
        end

        // Clear input valid after sending frame
        @(posedge clk);
        tx_valid_in <= 1'b0;
        tx_data_in  <= 1'b0;

        // Wait for both IFFT and FFT IP Core pipeline latencies
        #2500;
        $display("\n=======================================================");
        $display("   SIMULATION FINISHED ");
        $display("=======================================================\n");
        $finish;
    end

    // -------------------------------------------------------------
    // Transmitter IFFT Output Monitor
    // -------------------------------------------------------------
    always @(posedge clk) begin
        if (tx_ifft_out_tvalid) begin
            $display("[TX IFFT OUT] Time: %0t ns | Real [7:0]: %d | Imag [15:8]: %d | Last: %b", 
                     $time, 
                     $signed(tx_ifft_out_tdata[7:0]), 
                     $signed(tx_ifft_out_tdata[15:8]), 
                     tx_ifft_out_tlast);
        end
    end

    // -------------------------------------------------------------
    // Receiver FFT Output Monitor
    // -------------------------------------------------------------
    always @(posedge clk) begin
        if (rx_fft_out_tvalid) begin
            $display("[RX FFT  OUT] Time: %0t ns | Real_I: %d | Imag_Q: %d | Last: %b", 
                     $time, 
                     $signed(rx_fft_I_out), 
                     $signed(rx_fft_Q_out), 
                     rx_fft_out_tlast);
        end
    end
```
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

