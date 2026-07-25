# QPSK FORMATTER
```verilog
`timescale 1ns / 1ps

module qpsk_to_axis_formatter (
    input  wire        clk,
    input  wire        rst_n,
    input  wire [7:0]  qpsk_I,
    input  wire [7:0]  qpsk_Q,
    input  wire        mapper_valid,
    input  wire        ifft_ready,
    output reg  [15:0] m_axis_tdata,
    output reg         m_axis_tvalid,
    output reg         m_axis_tlast
);

    reg [2:0] sample_count;

    // Successful AXI-Stream handshake condition
    wire handshaken = m_axis_tvalid && ifft_ready;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            sample_count  <= 3'sd0;
            m_axis_tdata  <= 16'sd0;
            m_axis_tvalid <= 1'b0;
            m_axis_tlast  <= 1'b0;
        end else begin
            // Priority 1: Check if fresh data is coming from mapper
            if (mapper_valid && (!m_axis_tvalid || ifft_ready)) begin
                m_axis_tdata  <= {qpsk_Q, qpsk_I}; // Q [15:8], I [7:0]
                m_axis_tvalid <= 1'b1;
                
                // Assert tlast on the 8th sample (sample_count == 7)
                m_axis_tlast  <= (sample_count == 3'sd7);
                
                // Increment counter from 0 to 7
                sample_count  <= (sample_count == 3'sd7) ? 3'sd0 : sample_count + 1'b1;
            
            // Priority 2: Clear valid/tlast after successful handshake when no new data
            end else if (handshaken) begin
                m_axis_tvalid <= 1'b0;
                m_axis_tlast  <= 1'b0;
            end
        end
    end

endmodule
```
