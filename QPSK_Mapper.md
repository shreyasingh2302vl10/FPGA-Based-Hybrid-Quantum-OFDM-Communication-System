LOGIC FLOW
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/5de8cfa5e1573d9ff73a3215741e6499728f54b3/ChatGPT%20Image%20May%2025%2C%202026%2C%2002_56_28%20PM.png)


# CODE
```verilog
module qpsk_mapper (
    input  wire       clk,
    input  wire       rst_n,
    input  wire       data_in,
    input  wire       valid_in,
    output reg [1:0]  qpsk_I,
    output reg [1:0]  qpsk_Q,
    output reg        valid_out
);

    reg        bit_phase;
    reg        buffer_bit;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            bit_phase  <= 1'b0;
            buffer_bit <= 1'b0;
            qpsk_I     <= 2'b00;
            qpsk_Q     <= 2'b00;
            valid_out  <= 1'b0;
        end else begin
            if (valid_in) begin
                if (bit_phase == 1'b0) begin
                    buffer_bit <= data_in;
                    bit_phase  <= 1'b1;
                    valid_out  <= 1'b0;
                end else begin
                    bit_phase <= 1'b0;
                    valid_out <= 1'b1;
                    
                    qpsk_I <= (buffer_bit == 1'b0) ? 2'b01 : 2'b11;
                    qpsk_Q <= (data_in    == 1'b0) ? 2'b01 : 2'b11;
                end
            end else begin
                valid_out <= 1'b0;
            end
        end
    end
endmodule
```
