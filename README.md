# SMART-ENERGY-METER

**Abstract**
```
This project presents the design and simulation of a Smart Energy Meter using Verilog 
HDL. The system simulates voltage and current waveforms, computes instantaneous 
power (P = V × I), accumulates total energy consumption, and displays values through a 
seven-segment interface. The design includes modules such as an input generator, 
arithmetic computation unit, memory register, BCD converters, seven-segment decoders, 
and a testbench for verification. Simulation results in Vivado confirm the correct 
operation of the smart meter, demonstrating real-time measurement and display of 
electrical quantities. This project showcases modular digital design, arithmetic modeling, 
and memory-based energy calculation in a real-world application context.
```

**Introduction** 
```
Smart meters have become an essential part of modern electrical systems, enabling 
accurate monitoring of voltage, current, and energy consumption. They help consumers 
manage electricity usage, support utilities in automated billing, and contribute to energy 
conservation. A smart energy meter typically measures electrical parameters, computes 
power, and stores energy readings at regular intervals. 
The objective of this project is to design and simulate a complete Smart Energy Meter 
using Verilog HDL. The system generates simulated voltage and current, computes 
instantaneous power, stores accumulated energy values, and displays results on a 
sevensegment display. The design demonstrates key concepts such as dataflow modeling, 
arithmetic operations, memory logic, modular hardware design, and simulation-based 
verification using Vivado. The project reflects real-world engineering practice and the 
importance of efficient digital system design.
```

**Problem Statement**
```
The increasing demand for accurate and efficient energy monitoring in modern power 
systems requires the use of intelligent metering devices that can measure electrical 
parameters in real time. Traditional energy meters are limited in their ability to track 
instantaneous voltage, current, and power variations, making them unsuitable for 
automated billing and smart energy management. 
The objective of this project is to design and simulate a Smart Energy Meter using 
Verilog HDL that can: 
⚫ Generate simulated voltage and current waveforms, 
⚫ Compute instantaneous power using digital arithmetic logic, 
⚫ Accumulate total energy consumption over time, and 
⚫ Display power and current values using a seven-segment display interface.
```

**Block Diagram**

<img width="647" height="298" alt="image" src="https://github.com/user-attachments/assets/9a1fa0e3-7687-4e02-9d15-743b0f5e05a5" />

**Verilog Code** 
```
module input_module( 
input clk, output reg [15:0] 
voltage, output reg [15:0] 
current); 
always @(posedge clk) begin 
voltage <= 200 + ($random % 50); // 200–250 V 
current <= 5 + ($random % 3); 
end 
endmodule 
// 2. POWER COMPUTATION: P = V × I 
module power_compute( input [15:0] 
voltage, input [15:0] current, 
output [31:0] power); 
assign power = voltage * current; 
endmodule // 3. ENERGY 
ACCUMULATION 
module 
energy_memory( input clk, input 
[31:0] power, output reg [63:0] 
energy); 
parameter TIME_INTERVAL = 1; always 
@(posedge clk) 
// 5–7 A 
energy <= energy + (power * TIME_INTERVAL); 
endmodule 
// 
4. 
SEVEN
SEGMENT DECODER module  
seven_seg_decoder( input [3:0] 
digit, output reg [6:0] seg); 
always @(*) begin 
case (digit) 
4'd0: seg = 7'b1111110; 
4'd1: seg = 7'b0110000; 
4'd2: seg = 7'b1101101; 
4'd3: seg = 7'b1111001; 
4'd4: seg = 7'b0110011; 
4'd5: seg = 7'b1011011; 
4'd6: seg = 7'b1011111; 
4'd7: seg = 7'b1110000; 
4'd8: seg = 7'b1111111; 
4'd9: seg = 7'b1111011; 
default: seg = 7'b0000000; 
endcase 
end 
endmodule 
// 5. BCD CONVERTER — 4-digit FOR POWER 
module bcd_converter( 
input [31:0] value, 
output reg [3:0] d1, 
output reg [3:0] d2, 
output reg [3:0] d3, 
output reg [3:0] d4); 
integer temp; 
always @(*) begin 
temp = value; d1 = temp % 10; 
temp = temp / 10; d2 = temp % 10; 
temp = temp / 10; d3 = temp % 10; 
temp = temp / 10; d4 = temp % 10; 
end 
endmodule 
// 6. BCD CONVERTER — 2-digit FOR CURRENT 
module bcd2_converter( input [15:0] value, 
output reg [3:0] u, 
output reg [3:0] t); 
integer tmp; 
always @(*) begin 
tmp = value; 
u = tmp % 10; t = 
(tmp / 10) % 10; 
end 
endmodule 
// 7. CLOCK DIVIDER for Slow 7-seg Refresh (1ms approx.) 
module clk_divider( input clk, output reg slow_clk); 
reg [19:0] count = 0; always 
@(posedge clk) begin count  
<= count + 1; if (count == 
500000) begin 
slow_clk <= ~slow_clk; 
count <= 0; end 
end 
endmodule 
// 8. TOP MODULE – SMART METER 
module smart_meter_top( 
input clk, output 
[6:0] seg1, 
output [6:0] 
seg2, output 
[6:0] seg3, 
output [6:0] 
seg4, output 
[6:0] segC1, 
output [6:0] 
segC2, output 
[15:0] 
voltage_out, 
output [15:0] 
current_out, 
output [31:0] 
power_out); 
wire [15:0] V, I; wire 
[31:0] P; wire [63:0] E; 
wire slow_clk; wire 
[3:0] d1, d2, d3, d4; wire 
[3:0] cu, ct; 
// MODULE CONNECTIONS 
input_module U1(clk, V, I); 
power_compute 
energy_memory 
clk_divider 
U2(V, I, P); 
U3(clk, P, E); 
U4(clk, slow_clk); 
// POWER → BCD 
bcd_converter U5(P, d1, d2, d3, d4); 
// CURRENT → BCD 
bcd2_converter U6(I, cu, ct); // 
7-segment decoders 
seven_seg_decoder S1(d1, seg1); 
seven_seg_decoder S2(d2, seg2); 
seven_seg_decoder S3(d3, seg3); 
seven_seg_decoder S4(d4, seg4); 
seven_seg_decoder SC1(cu, 
segC1); seven_seg_decoder 
SC2(ct, segC2); // Waveform 
debug outputs assign 
voltage_out = V; assign  
current_out = I; assign 
power_out  = P;
endmodule
```

**TESTBENCH** 
 ```
`timescale 1ns/1ps module 
smart_meter_tb; reg clk; wire [6:0] 
seg1, seg2, seg3, seg4; wire [6:0] 
segC1, segC2; wire [15:0] 
voltage_out, current_out; wire [31:0] 
power_out; smart_meter_top DUT( 
.clk(clk), 
.seg1(seg1), .seg2(seg2), .seg3(seg3), .seg4(seg4), 
.segC1(segC1), .segC2(segC2), 
.voltage_out(voltage_out), 
.current_out(current_out), 
.power_out(power_out)); 
// Clock = 10ns period 
initial 
clk 
forever 
begin 
= 0; 
#5 
clk = ~clk; 
end 
initial begin 
$display("SMART METER SIM STARTED"); 
#5000; 
$finish; 
end 
endmodule
```

**Simulation Result**

<img width="1920" height="1080" alt="Screenshot 2025-11-18 164656" src="https://github.com/user-attachments/assets/9c533528-2841-4938-80fd-fbfaef919558" />

**Conclusion**
```
The Smart Energy Meter was successfully designed and simulated using Verilog HDL. All 
modules function as intended, demonstrating real-time voltage and current generation, 
correct power computation, energy accumulation, and proper seven-segment display 
output. The system is modular, efficient, and matches the real-world behavior of an 
energy-monitoring device. This project highlights digital system design, hierarchical 
modeling, arithmetic computation, and verification using Vivado simulation.
```
