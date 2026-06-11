```verilog
module qpsk_to_axis_formatter (
    input  wire clk,
    input  wire rst_n,
    input  wire [1:0] qpsk_I,
    input  wire [1:0] qpsk_Q,
    input  wire mapper_valid,
    input  wire ifft_ready,
    output reg  [15:0] m_axis_tdata,
    output reg  m_axis_tvalid,
    output reg  m_axis_tlast
);

    reg [2:0] sample_count;

    always @(posedge clk or negedge rst_n) begin
    if (!rst_n) begin
        sample_count <= 3'd0;
        m_axis_tdata <= 16'd0;
        m_axis_tvalid <= 1'b0;
        m_axis_tlast <= 1'b0;
    end else begin
        if (mapper_valid && ifft_ready) begin
            // Packing: Q (bits 15-8) and I (bits 7-0)
            m_axis_tdata <= {qpsk_Q, 6'b000000, qpsk_I, 6'b000000};
            m_axis_tvalid <= 1'b1;
            m_axis_tlast <= (sample_count == 3'd7);
            sample_count <= (sample_count == 3'd7) ? 3'd0 : sample_count + 1'b1;
        end else begin
            m_axis_tvalid <= 1'b0;
            m_axis_tlast <= 1'b0;
        end
    end
end
endmodule
```
