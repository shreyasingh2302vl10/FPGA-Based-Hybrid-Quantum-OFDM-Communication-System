# Heriarchy of Modules
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/2c6738483fbe499bc84b70610075f8f6ccd746c4/Screenshot%202026-06-12%20041422.png)
# Code of Transmitter Top Module
```verilog
module transmitter_top (
    input  wire clk,
    input  wire rst_n,
    input  wire data_in,
    input  wire valid_in,
    output wire [1:0] qpsk_I_out, 
    output wire [1:0] qpsk_Q_out,
    output wire [31:0] ifft_out_tdata, // FFT IP 8-point processed output
    output wire        ifft_out_tvalid
);
    wire [1:0] w_qpsk_I, w_qpsk_Q;
    wire w_mapper_valid;
    
    
    wire [31:0] w_axis_tdata; 
    wire w_axis_tvalid, w_axis_tlast, w_ifft_ready;

    
    wire [15:0] w_config_tdata = 16'h0006; 

    assign qpsk_I_out = w_qpsk_I;
    assign qpsk_Q_out = w_qpsk_Q;

    
    qpsk_mapper u_mapper (
        .clk(clk), 
        .rst_n(rst_n),
        .data_in(data_in), 
        .valid_in(valid_in),
        .qpsk_I(w_qpsk_I), 
        .qpsk_Q(w_qpsk_Q), 
        .valid_out(w_mapper_valid)
    );

    
    qpsk_to_axis_formatter u_formatter (
        .clk(clk), 
        .rst_n(rst_n),
        .qpsk_I(w_qpsk_I), 
        .qpsk_Q(w_qpsk_Q), 
        .mapper_valid(w_mapper_valid),
        .ifft_ready(w_ifft_ready),
        .m_axis_tdata(w_axis_tdata),    // Connected to 32-bit wire
        .m_axis_tvalid(w_axis_tvalid), 
        .m_axis_tlast(w_axis_tlast)
    );

    xfft_0 u_ifft (
        .aclk(clk),
                .s_axis_config_tdata(w_config_tdata), 
        .s_axis_config_tvalid(1'b1),            // Hamesha high taaki config set rahe
        
       
        .s_axis_data_tdata(w_axis_tdata), 
        
        .s_axis_data_tvalid(w_axis_tvalid),
        .s_axis_data_tready(w_ifft_ready),
        .s_axis_data_tlast(w_axis_tlast),
        
      
        .m_axis_data_tdata(ifft_out_tdata),
        .m_axis_data_tvalid(ifft_out_tvalid),
        .m_axis_data_tready(1'b1)               // Testbench is always ready to accept
    );

endmodule
```
