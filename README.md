# ATM_fsm
implementation of ATM using verilog
//rtl.v

module atm_fsm (
    input clk,
    input reset,
    input start,
    input pin_ok,
    input menu_valid,
    input [1:0] menu_select,
    input process_done,
    output reg [2:0] state
);

    localparam S0_IDLE          = 3'd0;
    localparam S1_PIN_CHECK     = 3'd1;
    localparam S2_MENU          = 3'd2;
    localparam S3_WITHDRAW      = 3'd3;
    localparam S4_DEPOSIT       = 3'd4;
    localparam S5_BALANCE_CHECK = 3'd5;

    reg [2:0] next_state;

    always @(posedge clk or posedge reset) begin
        if (reset)
            state <= S0_IDLE;
        else
            state <= next_state;
    end

    always @(*) begin
        next_state = state;

        case (state)

            S0_IDLE: begin
                if (start)
                    next_state = S1_PIN_CHECK;
                else
                    next_state = S0_IDLE;
            end

            S1_PIN_CHECK: begin
                if (pin_ok)
                    next_state = S2_MENU;
                else
                    next_state = S0_IDLE;
            end

            S2_MENU: begin
                if (menu_valid) begin
                    case (menu_select)
                        2'b00: next_state = S3_WITHDRAW;
                        2'b01: next_state = S4_DEPOSIT;
                        2'b10: next_state = S5_BALANCE_CHECK;
                        default: next_state = S0_IDLE;
                    endcase
                end
                else begin
                    next_state = S0_IDLE;
                end
            end

            S3_WITHDRAW: begin
                if (process_done)
                    next_state = S0_IDLE;
                else
                    next_state = S3_WITHDRAW;
            end

            S4_DEPOSIT: begin
                if (process_done)
                    next_state =S0_IDLE;
                else
                    next_state = S4_DEPOSIT ;
            end

            S5_BALANCE_CHECK: begin
                if (process_done)
                    next_state =S0_IDLE ;
                else
                    next_state = S5_BALANCE_CHECK;
            end

            default: begin
                next_state = S0_IDLE;
            end

        endcase
    end

endmodule


//testbench.v


module tb;

    reg clk;
    reg reset;
    reg start;
    reg pin_ok;
    reg menu_valid;
    reg [1:0] menu_select;
    reg process_done;

    wire [2:0] state;

    atm_fsm dut (
        .clk(clk),
        .reset(reset),
        .start(start),
        .pin_ok(pin_ok),
        .menu_valid(menu_valid),
        .menu_select(menu_select),
        .process_done(process_done),
        .state(state)
    );

    // Clock generation
    always #5 clk = ~clk;

    initial begin

        // Initial values
        clk = 0;
        reset = 1;
        start = 0;
        pin_ok = 0;
        menu_valid = 0;
        menu_select = 2'b00;
        process_done = 0;

        // Reset
        #10 reset = 0;

        //====================================
        // CONDITION 1: WITHDRAW
        //====================================

        #15 start = 1;
        #10 pin_ok = 1;
        #10 menu_valid = 1;
            menu_select = 2'b00;
        #10 process_done = 1;
        #10 process_done = 0;

        // Return inputs to default
        start = 0;
        pin_ok = 0;
        menu_valid = 0;

        //====================================
        // CONDITION 2: DEPOSIT
        //====================================

        #20 start = 1;
        #10 pin_ok = 1;
        #10 menu_valid = 1;
            menu_select = 2'b01;
        #10 process_done = 1;
        #10 process_done = 0;

        // Finish simulation
        #20 $finish;

    end

endmodule
