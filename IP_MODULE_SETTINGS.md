# CONFIGURATION SECTION
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/171fa5c0c799acf9319bcc0895387b0571e8821d/Screenshot%202026-06-12%20033718.png)
# IMPLEMENTATION SECTION
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/34b7df36662c03795667c2bd65c5be1c0c9b92c0/Screenshot%202026-06-12%20033726.png)
# IFFT CODE 
```scheme 
 xfft_0 u_ifft (
        .aclk(clk),
        .s_axis_config_tdata(8'b00000000), // '0' means Inverse FFT (IFFT)
        .s_axis_config_tvalid(1'b1),
        .s_axis_data_tdata(w_axis_tdata),
        .s_axis_data_tvalid(w_axis_tvalid),
        .s_axis_data_tready(w_ifft_ready),
        .s_axis_data_tlast(w_axis_tlast),
        .m_axis_data_tdata(ifft_out_tdata),
        .m_axis_data_tvalid(ifft_out_tvalid),
        .m_axis_data_tready(1'b1)
    );
    ```
