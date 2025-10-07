#  Verilog Lab 4 — Counters (DE10-Lite / Quartus Prime)

**Student:** Ali Behbehani  
**Board:** DE10-Lite (50 MHz clock)  
**Tool:** Quartus Prime Lite Edition  

---

##  Overview
This lab explores four ways to build counters in Verilog :

1. A structural 8-bit T-Flip-Flop counter  
2. A behavioral 16-bit counter (Q <= Q + 1)  
3. An LPM (lpm_counter) implementation  
4. A time-base display that counts 0→9 on HEX0 each second  

Each part was compiled, programmed, and verified on the DE10-Lite FPGA board.

---

##  Part 1 — Structural 8-bit T-Flip-Flop Counter
Implements an 8-bit counter using eight T-flip-flops and an asynchronous active-low clear.

```verilog
module TFlipFlop (
    input  T, clk, clr,
    output reg Q
);
  always @(posedge clk or negedge clr) begin
    if (!clr)
      Q <= 0;
    else if (T)
      Q <= ~Q;
  end
endmodule
```
![Alt text](images/rtl_viewer.png)



## Part 2 - Behavioral 16-bit Counter

```verilog
reg [15:0] Q;
always @(posedge MAX10_CLK1_50 or negedge SW[9]) begin
  if (!SW[9])
    Q <= 0;
  else if (SW[0])
    Q <= Q + 1;
end

seg7Decoder HEX0_u(Q[3:0],  HEX0);
seg7Decoder HEX1_u(Q[7:4],  HEX1);
seg7Decoder HEX2_u(Q[11:8], HEX2);
seg7Decoder HEX3_u(Q[15:12], HEX3);
```

## Part 3 - LPM Counter Module

```verilog
`default_nettype none

wire        w_clk = MAX10_CLK1_50;  // board clock
wire [15:0] w_Q;


lpm_counter #(
    .LPM_WIDTH     (16),
    .LPM_DIRECTION ("UP")
) U_CNT (
    .clock  (w_clk),
    .cnt_en (SW[0]),
    .sclr   (SW[9]),     
    .aclr   (1'b0),
    .q      (w_Q),
    .sload  (1'b0),
    .data   (16'd0),
    .updown (1'b1)
);

seg7Decoder HEX0_u (w_Q[3:0],    HEX0);
seg7Decoder HEX1_u (w_Q[7:4],    HEX1);
seg7Decoder HEX2_u (w_Q[11:8],   HEX2);
seg7Decoder HEX3_u (w_Q[15:12],  HEX3);
```

## Part 4 - One-Second Digit Display
```verilog
`default_nettype none

wire rst_n = ~KEY[0];

localparam integer TICKS_PER_SEC = 50_000_000;
reg [25:0] cnt;

always @(posedge MAX10_CLK1_50 or negedge rst_n) begin
  if (!rst_n)
    cnt <= 26'd0;
  else if (cnt == TICKS_PER_SEC-1)
    cnt <= 26'd0;
  else
    cnt <= cnt + 26'd1;
end

wire tick = (cnt == TICKS_PER_SEC-1);

reg [3:0] digit;
always @(posedge MAX10_CLK1_50 or negedge rst_n) begin
  if (!rst_n)
    digit <= 4'd0;
  else if (tick)
    digit <= (digit == 4'd9) ? 4'd0 : digit + 4'd1;
end

seg7Decoder h0(.i_bin(digit), .o_HEX(HEX0));

```

## Conclusion 


Although it requires more resources, structural design (flip-flops) demonstrates the fundamentals.
The behavioral design (Q <= Q + 1) is straightforward and useful.
The most resource-efficient counter is the LPM counter.
Real-world designs like scrolling displays and digit tickers can use counters.














