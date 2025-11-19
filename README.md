# Energy Efficient Coach Controller

## Aim
To design and simulate an energy-efficient controller using Verilog HDL that automatically switches ON coach lights and fans when a passenger is present and switches them OFF when the coach becomes empty using an FSM and delay timer.

## Apparatus Required
 * Vivado 2024.1

## Theory
The Energy-Efficient Coach Controller works on the principle of a Finite State Machine (FSM).
The system has three states:

* IDLE (00) – No passenger present; lights and fan OFF

* ACTIVE (01) – Passenger present; lights and fan ON

* WAIT (10) – Passenger left; timer runs before turning OFF

A sensor input (passenger_in) detects the presence of a passenger.
When the passenger exits, the controller enters the WAIT state where a delay timer counts clock cycles. If no passenger returns before the timer expires, the system switches back to IDLE and turns OFF the lights and fan.

This method saves power and prevents frequent ON/OFF switching.
The design uses Verilog concepts such as sequential logic, combinational logic, conditional statements, and timing blocks.

## Block Diagram
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/852fe388-bad3-4c09-ba3d-ce1736b7bb62" /> 

## Verilog Code
```verilog
module coach_controller (
    input  wire clk,
    input  wire reset,
    input  wire passenger_in,      
   output reg  lights,
    output reg  fan,
output reg  [1:0] state_out   
);
    localparam IDLE  = 2'b00;
    localparam ACTIVE = 2'b01;
    localparam WAIT   = 2'b10;
    reg [1:0] state, next_state;
    reg [7:0] timer;           
parameter DELAY = 5;       
    always @(posedge clk or posedge reset) begin
        if (reset) begin
            state <= IDLE;
            timer <= 0;
        end else begin
            state <= next_state;
            if (state == WAIT)
                timer <= timer + 1;
            else
                timer <= 0;
        end
    end
  
always @(*) begin
        case (state)
            IDLE: begin
                if (passenger_in) next_state = ACTIVE;
                else              
next_state = IDLE;
            end
            ACTIVE: begin
                if (!passenger_in) next_state = WAIT;
                else               
next_state = ACTIVE;
            end
            WAIT: begin
                if (passenger_in)        next_state = ACTIVE;
                else if (timer >= DELAY) next_state = IDLE;
                else                    
 next_state = WAIT;
            end
            default: next_state = IDLE;
        endcase
    end
    always @(*) begin
        case (state)
            IDLE: begin
                lights = 0;
                fan    = 0;
            end
          
WAIT,
            ACTIVE: begin
                lights = 1;
                fan    = 1;
            end
        endcase   
   state_out = state;
    end
Endmodule
```
## Test Bench
```
`timescale 1ns/1ps
module tb_controller;
    reg clk, reset, passenger_in;
    wire lights, fan;
    wire [1:0] state_out;
    coach_controller DUT (
        .clk(clk),
        .reset(reset),
        .passenger_in(passenger_in),
        .lights(lights),
        .fan(fan),
        .state_out(state_out)  );
    always #5 clk = ~clk;  
initial begin
        clk = 0;  
  reset = 1;
        passenger_in = 0;  
 #20 reset = 0;
        #30 passenger_in = 1;
        #100 passenger_in = 0;  
        #200;
        passenger_in = 1;
        #50 passenger_in = 0;
        #200;
        $finish;
    end
Endmodule
```
## Simulation Output
<img width="1920" height="1080" alt="Screenshot 2025-11-18 154426" src="https://github.com/user-attachments/assets/5050eaee-3df9-4ff8-b5e0-7887b92279a6" /> 

## Result
The FSM-based coach controller successfully detected passenger presence and automatically controlled the lights and fan. The system operated accurately with timed shutoff, achieving energy-efficient performance.
