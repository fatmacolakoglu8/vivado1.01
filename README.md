# vivado1.01

module Arbiter(
    input clk, rst, ra, rb,        // ra = request from requestor a, rb = request from requestor b, level signals
    input [1:0] PA, PB,           // priority values for requestor a and requestor b, unsigned, higher number is higher priority
    output reg ga, gb             // ga = grant to a, gb = grant to b
);
    // FSM states
    localparam [1:0] state_WaitReq = 0, 
                     state_GrantA  = 1, 
                     state_GrantB  = 2;

    reg [1:0] state; // Current state

    // FSM Logic
    always @(posedge clk or posedge rst) begin
        if (rst) begin
            state <= state_WaitReq;
        end else begin
            case (state)
                state_WaitReq: begin
                    if (ra && !rb) begin
                        state <= state_GrantA; 
                    end 
                    if (!ra && rb) begin
                        state <= state_GrantB; 
                    end 
                    if (ra && rb) begin
                        if (PB > PA) begin
                            state <= state_GrantB; 
                        end else begin
                            state <= state_GrantA; 
                        end
                    end
                end
                state_GrantA: begin
                    if (!ra) begin
                        state <= state_WaitReq; 
                    end
                end
                state_GrantB: begin
                    if (!rb) begin
                        state <= state_WaitReq; 
                    end
                end
            endcase
        end
    end

    // Output Logic
    always @(*) begin
        case (state)
            state_WaitReq: begin
                ga = 0;
                gb = 0;
            end
            state_GrantA: begin
                ga = 1;
                gb = 0;
            end
            state_GrantB: begin
                ga = 0;
                gb = 1;
            end
        endcase
    end
endmodule
