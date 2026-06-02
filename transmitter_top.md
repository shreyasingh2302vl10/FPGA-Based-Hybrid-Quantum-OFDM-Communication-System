`timescale 1ns / 1ps

module transmitter_top_with_ifft_8point (
    input wire clk,
    input wire rst_n,
    input wire data_in,
    input wire valid_in,
    output wire ifft_out_valid,
    output wire [31:0] ifft_out_data,
    output wire ifft_out_last
);

    // Internal wires
    wire [1:0] w_qpsk_I, w_qpsk_Q;
    wire w_mapper_valid;
    wire [7:0] w_parallel_I, w_parallel_Q;
    wire w_parallel_valid;

    // 1. QPSK Mapper
    qpsk_mapper u_qpsk_mapper (
        .clk(clk), .rst_n(rst_n), .data_in(data_in), .valid_in(valid_in),
        .qpsk_I(w_qpsk_I), .qpsk_Q(w_qpsk_Q), .valid_out(w_mapper_valid)
    );

    // 2. Serial to Parallel (Updated to 8 subcarriers)
    serial_to_parallel #(.NUM_SUBCARRIERS(8), .BIT_WIDTH(2)) u_serial_to_parallel (
        .clk(clk), .rst_n(rst_n), .in_I(w_qpsk_I), .in_Q(w_qpsk_Q),
        .valid_in(w_mapper_valid), .parallel_I(w_parallel_I), .parallel_Q(w_parallel_Q),
        .parallel_valid(w_parallel_valid)
    );

    // 3. Data Mapping & tlast generation (Updated for 8-point)
    reg [15:0] ip_in_tdata;
    reg        ip_in_tvalid;
    reg        ip_in_tlast;
    reg [2:0]  counter;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            ip_in_tvalid <= 0;
            ip_in_tdata  <= 0;
            ip_in_tlast  <= 0;
            counter      <= 0;
        end else if (w_parallel_valid) begin
            ip_in_tdata  <= {w_parallel_Q, w_parallel_I}; 
            ip_in_tvalid <= 1;
            
            // Fixed for 8-point IFFT
            if (counter == 3'd7) begin 
                ip_in_tlast <= 1;
                counter <= 0;
            end else begin
                ip_in_tlast <= 0;
                counter <= counter + 1;
            end
        end else begin
            ip_in_tvalid <= 0;
            ip_in_tlast  <= 0;
        end
    end

    // 4. IFFT IP Core
    xfft_0 u_xfft_core (
        .aclk(clk), 
        .aresetn(rst_n),
        .s_axis_config_tdata(16'd1),  // Force Inverse FFT
        .s_axis_config_tvalid(1'b1),
        .s_axis_data_tdata(ip_in_tdata),
        .s_axis_data_tvalid(ip_in_tvalid),
        .s_axis_data_tready(), // Open port
        .s_axis_data_tlast(ip_in_tlast),
        .m_axis_data_tdata(ifft_out_data),
        .m_axis_data_tvalid(ifft_out_valid),
        .m_axis_data_tlast(ifft_out_last)
    );

endmodule
