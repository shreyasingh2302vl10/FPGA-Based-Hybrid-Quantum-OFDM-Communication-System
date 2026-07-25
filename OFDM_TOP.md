# CIRCUIT DESIGN
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/31304e5d0ad2faeb51c7013e28ecc7ff91276337/Circuit_Design.png)
# CODE-Top Module
```verilog
`timescale 1ns / 1ps

module ofdm_top (
    input  wire        clk,
    input  wire        rst_n,
    
    // --- Transmitter Inputs ---
    input  wire        tx_data_in,
    input  wire        tx_valid_in,
    
    // --- Transmitter IFFT Output ---
    output wire [15:0] tx_ifft_out_tdata,
    output wire        tx_ifft_out_tvalid,
    output wire        tx_ifft_out_tlast,
    
    // --- Receiver FFT Output (Combined 16-bit: [15:8]=Q, [7:0]=I) ---
    output wire [15:0] rx_fft_out_tdata,
    output wire        rx_fft_out_tvalid,
    output wire        rx_fft_out_tlast,
    
    // --- Parsed Receiver Outputs (I/Q Split) ---
    output wire [7:0]  rx_fft_I_out,
    output wire [7:0]  rx_fft_Q_out,

    // --- Demapper Outputs (2-bit Symbol Interface) ---
    output wire [7:0]  mapped_I_out,
    output wire [7:0]  mapped_Q_out,
    output wire [1:0]  rx_demap_2bit_out, // Updated: [1]=I bit, [0]=Q bit
    output wire        rx_demap_valid
);

    // =============================================================
    // 1. Transmitter Instance (Tx Top) - Uses xfft_0 (IFFT) inside
    // =============================================================
    wire [7:0]  w_tx_qpsk_I, w_tx_qpsk_Q;
    wire [15:0] w_tx_ifft_tdata;
    wire        w_tx_ifft_tvalid;
    wire        w_tx_ifft_tlast;

    transmitter_top u_transmitter (
        .clk             (clk),
        .rst_n           (rst_n),
        .data_in         (tx_data_in),
        .valid_in        (tx_valid_in),
        .qpsk_I_out      (w_tx_qpsk_I),
        .qpsk_Q_out      (w_tx_qpsk_Q),
        .ifft_out_tdata  (w_tx_ifft_tdata),
        .ifft_out_tvalid (w_tx_ifft_tvalid),
        .ifft_out_tlast  (w_tx_ifft_tlast)
    );

    // Drive Top-level Transmitter Outputs
    assign tx_ifft_out_tdata  = w_tx_ifft_tdata;
    assign tx_ifft_out_tvalid = w_tx_ifft_tvalid;
    assign tx_ifft_out_tlast  = w_tx_ifft_tlast;

    // =============================================================
    // 2. Direct Loopback Channel (Tx IFFT -> Rx FFT)
    // =============================================================
    wire [15:0] w_rx_channel_tdata  = w_tx_ifft_tdata;
    wire        w_rx_channel_tvalid = w_tx_ifft_tvalid;
    wire        w_rx_channel_tlast  = w_tx_ifft_tlast;

    // =============================================================
    // 3. Receiver FFT Setup (Using xfft_1 Instance)
    // =============================================================
    wire [15:0] w_rx_fft_tdata;
    wire        w_rx_fft_tvalid;
    wire        w_rx_fft_tlast;
    wire        w_rx_fft_ready;

    // Configuration TDATA for Receiver (8'h01 = Forward FFT)
    wire [7:0]  w_rx_config_tdata = 8'h01; 
    wire        w_rx_config_tready;
    reg         r_rx_config_tvalid;

    // Safe Configuration Handshake Sequence
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            r_rx_config_tvalid <= 1'b1;
        end else if (r_rx_config_tvalid && w_rx_config_tready) begin
            r_rx_config_tvalid <= 1'b0;
        end
    end

    // Xilinx FFT IP Instance #2 (Receiver Core: xfft_1)
    xfft_1 u_rx_fft (
        .aclk                        (clk),
        
        // Configuration Channel
        .s_axis_config_tdata         (w_rx_config_tdata),
        .s_axis_config_tvalid        (r_rx_config_tvalid),
        .s_axis_config_tready        (w_rx_config_tready),
        
        // Data Input Channel (Loopback from Tx)
        .s_axis_data_tdata           (w_rx_channel_tdata),
        .s_axis_data_tvalid          (w_rx_channel_tvalid),
        .s_axis_data_tready          (w_rx_fft_ready),
        .s_axis_data_tlast           (w_rx_channel_tlast),
        
        // Data Output Channel
        .m_axis_data_tdata           (w_rx_fft_tdata),
        .m_axis_data_tvalid          (w_rx_fft_tvalid),
        .m_axis_data_tready          (1'b1),
        .m_axis_data_tlast           (w_rx_fft_tlast),

        // Event Ports
        .event_frame_started         (),
        .event_tlast_unexpected      (),
        .event_tlast_missing         (),
        .event_status_channel_halt   (),
        .event_data_in_channel_halt  (),
        .event_data_out_channel_halt ()
    );

    // =============================================================
    // 4. Receiver Output Mapping & QPSK Demapper Instantiation
    // =============================================================
    assign rx_fft_out_tdata  = w_rx_fft_tdata;
    assign rx_fft_out_tvalid = w_rx_fft_tvalid;
    assign rx_fft_out_tlast  = w_rx_fft_tlast;

    // Direct Extraction: [7:0] Real (I) and [15:8] Imaginary (Q)
    assign rx_fft_I_out      = w_rx_fft_tdata[7:0];
    assign rx_fft_Q_out      = w_rx_fft_tdata[15:8];

    // QPSK Demapper Instance
    qpsk_demapper u_demapper (
        .clk            (clk),
        .rst_n          (rst_n),
        .fft_tdata_in   (rx_fft_out_tdata),
        .fft_tvalid_in  (rx_fft_out_tvalid),
        .mapped_I_out   (mapped_I_out),
        .mapped_Q_out   (mapped_Q_out),
        .demap_2bit_out (rx_demap_2bit_out), // Paired 2-bit symbol output
        .demap_valid    (rx_demap_valid)
    );

endmodule
```
