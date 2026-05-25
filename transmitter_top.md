```verilog
`timescale 1ns / 1ps

module transmitter_top (
    input wire clk,
    input wire rst_n,
    input wire data_in,
    input wire valid_in,
    output wire [15:0] tx_out0_r, output wire [15:0] tx_out0_i,
    output wire [15:0] tx_out1_r, output wire [15:0] tx_out1_i,
    output wire [15:0] tx_out2_r, output wire [15:0] tx_out2_i,
    output wire [15:0] tx_out3_r, output wire [15:0] tx_out3_i,
    output wire [15:0] tx_out4_r, output wire [15:0] tx_out4_i,
    output wire tx_valid_out
);

    wire signed [1:0] w_qpsk_I;
    wire signed [1:0] w_qpsk_Q;
    wire w_mapper_valid;

    wire signed [7:0] w_parallel_I;
    wire signed [7:0] w_parallel_Q;
    wire w_s2p_valid;

    reg signed [15:0] r_X0_r, r_X0_i;
    reg signed [15:0] r_X1_r, r_X1_i;
    reg signed [15:0] r_X2_r, r_X2_i;
    reg signed [15:0] r_X3_r, r_X3_i;
    reg r_ifft_start;

    wire signed [15:0] w_x0_r, w_x0_i;
    wire signed [15:0] w_x1_r, w_x1_i;
    wire signed [15:0] w_x2_r, w_x2_i;
    wire signed [15:0] w_x3_r, w_x3_i;

    qpsk_mapper u_qpsk_mapper (
        .clk(clk),
        .rst_n(rst_n),
        .data_in(data_in),
        .valid_in(valid_in),
        .qpsk_I(w_qpsk_I),
        .qpsk_Q(w_qpsk_Q),
        .valid_out(w_mapper_valid)
    );

    serial_to_parallel #(
        .NUM_SUBCARRIERS(4),
        .BIT_WIDTH(2)
    ) u_serial_to_parallel (
        .clk(clk),
        .rst_n(rst_n),
        .in_I(w_qpsk_I),
        .in_Q(w_qpsk_Q),
        .valid_in(w_mapper_valid),
        .parallel_I(w_parallel_I),
        .parallel_Q(w_parallel_Q),
        .parallel_valid(w_s2p_valid)
    );

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            r_X0_r <= 16'h0000; r_X0_i <= 16'h0000;
            r_X1_r <= 16'h0000; r_X1_i <= 16'h0000;
            r_X2_r <= 16'h0000; r_X2_i <= 16'h0000;
            r_X3_r <= 16'h0000; r_X3_i <= 16'h0000;
            r_ifft_start <= 1'b0;
        end else begin
            r_ifft_start <= w_s2p_valid;
            if (w_s2p_valid) begin
                r_X0_r <= (w_parallel_I[1:0] == 2'b01) ? 16'h2000 : 16'hE000;
                r_X0_i <= (w_parallel_Q[1:0] == 2'b01) ? 16'h2000 : 16'hE000;

                r_X1_r <= (w_parallel_I[3:2] == 2'b01) ? 16'h2000 : 16'hE000;
                r_X1_i <= (w_parallel_Q[3:2] == 2'b01) ? 16'h2000 : 16'hE000;

                r_X2_r <= (w_parallel_I[5:4] == 2'b01) ? 16'h2000 : 16'hE000;
                r_X2_i <= (w_parallel_Q[5:4] == 2'b01) ? 16'h2000 : 16'hE000;

                r_X3_r <= (w_parallel_I[7:6] == 2'b01) ? 16'h2000 : 16'hE000;
                r_X3_i <= (w_parallel_Q[7:6] == 2'b01) ? 16'h2000 : 16'hE000;
            end
        end
    end

    ifft_top u_ifft_top (
        .clk(clk),
        .rst(!rst_n),
        .X0_r(r_X0_r), .X0_i(r_X0_i),
        .X1_r(r_X1_r), .X1_i(r_X1_i),
        .X2_r(r_X2_r), .X2_i(r_X2_i),
        .X3_r(r_X3_r), .X3_i(r_X3_i),
        .x0_r(w_x0_r), .x0_i(w_x0_i),
        .x1_r(w_x1_r), .x1_i(w_x1_i),
        .x2_r(w_x2_r), .x2_i(w_x2_i),
        .x3_r(w_x3_r), .x3_i(w_x3_i)
    );

    reg [1:0] r_valid_pipe;
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            r_valid_pipe <= 2'b00;
        end else begin
            r_valid_pipe <= {r_valid_pipe[0], r_ifft_start};
        end
    end
    assign tx_valid_out = r_valid_pipe[1];

    add_cyclic_prefix u_add_cyclic_prefix (
        .clk(clk),
        .rst(!rst_n),
        .x0_r(w_x0_r), .x0_i(w_x0_i),
        .x1_r(w_x1_r), .x1_i(w_x1_i),
        .x2_r(w_x2_r), .x2_i(w_x2_i),
        .x3_r(w_x3_r), .x3_i(w_x3_i),
        .cp_out0_r(tx_out0_r), .cp_out0_i(tx_out0_i),
        .cp_out1_r(tx_out1_r), .cp_out1_i(tx_out1_i),
        .cp_out2_r(tx_out2_r), .cp_out2_i(tx_out2_i),
        .cp_out3_r(tx_out3_r), .cp_out3_i(tx_out3_i),
        .cp_out4_r(tx_out4_r), .cp_out4_i(tx_out4_i)
    );

endmodule
```
