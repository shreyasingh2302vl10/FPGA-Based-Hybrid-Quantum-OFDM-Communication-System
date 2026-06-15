# FFT_Block Settings
### BEFORE CP
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/38dd775c39bbe0032a09fcd033ca84799710c27a/Without_Cyclic_Prefix.png
)
### AFTER CP
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/38dd775c39bbe0032a09fcd033ca84799710c27a/Cyclic_Prefix.png)
### OUTPUT WITH CP 
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/f6234f1dfc9a161d867100f0384bea3f3d584836/CP_output.jpeg)
# Transmitter Modified Code 
``` scheme 
module transmitter_top (
    input  wire clk,
    input  wire rst_n,
    input  wire data_in,
    input  wire valid_in,
    output wire [1:0] qpsk_I_out, 
    output wire [1:0] qpsk_Q_out,
    output wire [31:0] ifft_out_tdata, // CP ke sath output
    output wire        ifft_out_tvalid
);
    wire [1:0] w_qpsk_I, w_qpsk_Q;
    wire w_mapper_valid;
    wire [15:0] w_axis_tdata;
    wire w_axis_tvalid, w_axis_tlast, w_ifft_ready;

    // CP Config: 
    // Bit 0: Forward (0) / Inverse (1) - Yahan IFFT ke liye 1 hona chahiye
    // Bits 1-8: CP Length (Aapki requirement ke hisaab se set karein)
    wire [15:0] w_config_tdata = {8'd16, 1'b1}; // Example: CP length 16, IFFT mode

    assign qpsk_I_out = w_qpsk_I;
    assign qpsk_Q_out = w_qpsk_Q;

    qpsk_mapper u_mapper (
        .clk(clk), .rst_n(rst_n),
        .data_in(data_in), .valid_in(valid_in),
        .qpsk_I(w_qpsk_I), .qpsk_Q(w_qpsk_Q), .valid_out(w_mapper_valid)
    );

    qpsk_to_axis_formatter u_formatter (
        .clk(clk), .rst_n(rst_n),
        .qpsk_I(w_qpsk_I), .qpsk_Q(w_qpsk_Q), .mapper_valid(w_mapper_valid),
        .ifft_ready(w_ifft_ready),
        .m_axis_tdata(w_axis_tdata), .m_axis_tvalid(w_axis_tvalid), .m_axis_tlast(w_axis_tlast)
    );

    xfft_0 u_ifft (
        .aclk(clk),
        .s_axis_config_tdata(w_config_tdata), 
        .s_axis_config_tvalid(1'b1),
        .s_axis_data_tdata(w_axis_tdata),
        .s_axis_data_tvalid(w_axis_tvalid),
        .s_axis_data_tready(w_ifft_ready),
        .s_axis_data_tlast(w_axis_tlast),
        .m_axis_data_tdata(ifft_out_tdata),
        .m_axis_data_tvalid(ifft_out_tvalid),
        .m_axis_data_tready(1'b1)
    );
endmodule
```
