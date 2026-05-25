# LOGIC FLOW
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/5cac408fd44f8d4a473fe50e0d6765b26eadfad7/Serial2Parallel.png)
# CODE
```verilog
module serial_to_parallel #(
    parameter NUM_SUBCARRIERS = 4,
    parameter BIT_WIDTH = 2
)(
    input  wire                               clk,
    input  wire                               rst_n,
    input  wire [BIT_WIDTH-1:0]               in_I,
    input  wire [BIT_WIDTH-1:0]               in_Q,
    input  wire                               valid_in,
    
    output reg  [(NUM_SUBCARRIERS*BIT_WIDTH)-1:0] parallel_I,
    output reg  [(NUM_SUBCARRIERS*BIT_WIDTH)-1:0] parallel_Q,
    output reg                                   parallel_valid
);

    reg [BIT_WIDTH-1:0] mem_I [0:NUM_SUBCARRIERS-1];
    reg [BIT_WIDTH-1:0] mem_Q [0:NUM_SUBCARRIERS-1];
    
    integer count;
    integer idx;

    always @(posedge clk or negedge rst_n)
     begin
                        if (!rst_n)
                            begin
                                        count          <= 0;
                                        parallel_I     <= 0;
                                        parallel_Q     <= 0;
                                        parallel_valid <= 1'b0;
                                        for(idx=0; idx<NUM_SUBCARRIERS; idx=idx+1)
                                            begin
                                                    mem_I[idx] <= 0;
                                                    mem_Q[idx] <= 0;
                                            end
                              end
                            else
                                    begin
                                            if (valid_in)
                                                    begin
                                                                    mem_I[count] <= in_I;
                                                                    mem_Q[count] <= in_Q;
                                                                    
                                                                    if (count == NUM_SUBCARRIERS - 1) 
                                                                            begin
                                                                                count  <= 0;
                                                                                parallel_valid <= 1'b1;
                                                                                
                                                                                for(idx=0; idx<NUM_SUBCARRIERS; idx=idx+1) 
                                                                                    begin
                                                                                            parallel_I[idx*BIT_WIDTH +: BIT_WIDTH] <= (idx == NUM_SUBCARRIERS - 1) ? in_I : mem_I[idx];
                                                                                            parallel_Q[idx*BIT_WIDTH +: BIT_WIDTH] <= (idx == NUM_SUBCARRIERS - 1) ? in_Q : mem_Q[idx];
                                                                                    end
                                                                            end 
                                                                    else 
                                                                            begin
                                                                                count  <= count + 1;
                                                                                parallel_valid <= 1'b0;
                                                                            end
                                                    end
                                            else 
                                                  begin
                                                        parallel_valid <= 1'b0;
                                                  end
                                    end
    end
endmodule
```
