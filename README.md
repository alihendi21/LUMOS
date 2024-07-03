## Iran Univeristy of Science and Technology
## Assignment : LUMOS RISC-V

- Name: seyed ali hendi
- Team Members: seyed ali hendi , amirhossein eslami , payam karimnejad
- Student ID: 99413262
- Date: 2024/05/25

## Fixed-Point Unit (FPU) Module:
This repository contains the Verilog implementation of a Fixed-Point Unit (FPU) capable of performing various arithmetic operations such as addition, subtraction, multiplication, and square root calculations. The module is parameterized to handle fixed-point numbers of configurable width and fractional bits.
## Overview:
The FPU module performs arithmetic operations on fixed-point numbers. It supports the following operations:

-Addition
-Subtraction
-Multiplication
-Square Root
## Parameters
WIDTH: Defines the bit-width of the operands and results. Default is 32 bits.
FBITS: Specifies the number of fractional bits in the fixed-point representation. Default is 10 bits.
## Inputs
clk: Clock signal used to synchronize operations.
reset: Reset signal to initialize or reset the module.
operand_1: First input operand, with a bit-width defined by WIDTH.
operand_2: Second input operand, with the same bit-width as operand_1.
operation: 2-bit signal used to select the desired arithmetic operation (addition, subtraction, multiplication, or square root).
## Outputs
result: Output register that stores the result of the arithmetic operation.
ready: Flag indicating whether the result is ready to be read.
## Module Declaration and Parameters :
```
`include "Defines.vh"

module Fixed_Point_Unit 
#(
    parameter WIDTH = 32,
    parameter FBITS = 10
)
(
    input wire clk,
    input wire reset,
    
    input wire [WIDTH - 1 : 0] operand_1,
    input wire [WIDTH - 1 : 0] operand_2,
    
    input wire [ 1 : 0] operation,

    output reg [WIDTH - 1 : 0] result,
    output reg ready
);
```
## Combinational Logic for Arithmetic Operations
The combinational logic block computes the result based on the selected operation.
```
    always @(*)
    begin
        case (operation)
            `FPU_ADD    : begin result = operand_1 + operand_2; ready = 1; end
            `FPU_SUB    : begin result = operand_1 - operand_2; ready = 1; end
            `FPU_MUL    : begin result = product[WIDTH + FBITS - 1 : FBITS]; ready = product_ready; end
            `FPU_SQRT   : begin result = root; ready = root_ready; end
            default     : begin result = 'bz; ready = 0; end
        endcase
    end
```
Addition and Subtraction: Simple arithmetic operations.
Multiplication: Uses a separate product signal and ready flag.
Square Root: Uses intermediate signals for square root calculation.
Default: Sets result and ready to high-impedance state ('bz) if the operation is not recognized.
## Reset Logic
The reset block resets the ready signal on a positive edge of the reset signal.
```
    always @(posedge reset)
    begin
        if (reset)  ready = 0;
        else        ready = 'bz;
    end
```
## Square Root Circuit
Registers and Signals
 ```   
    reg [WIDTH - 1 : 0] root;
    reg root_ready;

    reg [1 : 0] square_root_stage;
    reg [1 : 0] next_square_root_stage;
 ```   
root: Holds the current approximation of the square root.
root_ready: Indicates when the square root computation is complete and root is valid.
square_root_stage and next_square_root_stage: Control the stages of the square root calculation.
## State Machine for Square Root Calculation
```
    always @(posedge clk) 
    begin
        if (operation == `FPU_SQRT) square_root_stage <= next_square_root_stage;
        else                        
        begin
            square_root_stage <= 2'b00;
            root_ready <= 0;
        end
    end 
```
Purpose: This block manages the state transitions for the square root calculation based on the operation input.
Behavior: When operation is FPU_SQRT, it advances through different stages (square_root_stage). Otherwise, it resets to the initial state (2'b00) and clears root_ready.
## State Transition Logic
```
    always @(*) 
    begin
        next_square_root_stage <= 'bz;
        case (square_root_stage)
            2'b00 : begin sqrt_start <= 0; next_square_root_stage <= 2'b01; end
            2'b01 : begin sqrt_start <= 1; next_square_root_stage <= 2'b10; end
            2'b10 : begin sqrt_start <= 0; next_square_root_stage <= 2'b10; end
        endcase    
    end
```
Purpose: Determines the next stage (next_square_root_stage) based on the current stage (square_root_stage).
Behavior: It progresses through stages (2'b00 to 2'b01 to 2'b10) by setting sqrt_start appropriately.
## Square Root Calculation Process

  
 ```   
    always @(posedge clk) 
    begin
        if (sqrt_start)
        begin
            sqrt_busy <= 1;
            root_ready <= 0;
            i <= 0;
            q <= 0;
            {ac, x} <= {{WIDTH{1'b0}}, operand_1, 2'b0};
        end

        else if (sqrt_busy)
        begin
            if (i == ITER-1) 
            begin  // we're done
                sqrt_busy <= 0;
                root_ready <= 1;
                root <= q_next;
            end

            else 
            begin  // next iteration
                i <= i + 1;
                x <= x_next;
                ac <= ac_next;
                q <= q_next;
                root_ready <= 0;
            end
        end
    end
```
Purpose: This block performs the iterative square root calculation when sqrt_start is active (sqrt_start is set in the state machine logic).
Behavior:
Initializes necessary variables (sqrt_busy, i, q, ac, x) when sqrt_start is asserted.
Iteratively computes the square root using a Newton-Raphson method:
Updates x, ac, and q for each iteration (i increases).
Stops when i reaches ITER-1 and sets root to q_next.
Signals root_ready when the square root computation is complete.
## Multiplication Circuit
Registers and Signals
  ``` 
    reg [64 - 1 : 0] product;
    reg product_ready;

    reg     [15 : 0] multiplierCircuitInput1;
    reg     [15 : 0] multiplierCircuitInput2;
    wire    [31 : 0] multiplierCircuitResult;

    Multiplier multiplier_circuit
    (
        .operand_1(multiplierCircuitInput1),
        .operand_2(multiplierCircuitInput2),
        .product(multiplierCircuitResult)
    );

    reg     [31 : 0] partialProduct1;
    reg     [31 : 0] partialProduct2;
    reg     [31 : 0] partialProduct3;
    reg     [31 : 0] partialProduct4;

    reg [2 : 0] multiplication_stage;
    reg [2 : 0] next_multiplication_stage;
```
product and product_ready: Hold the result and signal completion for the multiplication operation.
multiplierCircuitInput1 and multiplierCircuitInput2: Inputs to the multiplier circuit.
multiplierCircuitResult: Output from the multiplier circuit (Multiplier module).
partialProduct1, partialProduct2, partialProduct3, partialProduct4: Hold intermediate results of multiplication.
multiplication_stage and next_multiplication_stage: Manage stages of the multiplication process.

## State Machine for Multiplication
```
    always @(posedge clk) 
    begin
        if (operation == `FPU_MUL)  multiplication_stage <= next_multiplication_stage;
        else                        multiplication_stage <= 'b0;
    end
```
Purpose: Controls the state transitions for the multiplication operation based on the operation input.
## Multiplication Stages Logic
```
    always @(*) 
    begin
        next_multiplication_stage <= 'bz;
        case (multiplication_stage)
            3'b000 :
            begin
                product_ready <= 0;

                multiplierCircuitInput1 <= 'bz;
                multiplierCircuitInput2 <= 'bz;

                partialProduct1 <= 'bz;
                partialProduct2 <= 'bz;
                partialProduct3 <= 'bz;
                partialProduct4 <= 'bz;

                next_multiplication_stage <= 3'b001;
            end 
            3'b001 : 
            begin
                multiplierCircuitInput1 <= operand_1[15 : 0];
                multiplierCircuitInput2 <= operand_2[15 : 0];
                partialProduct1 <= multiplierCircuitResult;
                next_multiplication_stage <= 3'b010;
            end
            3'b010 : 
            begin
                multiplierCircuitInput1 <= operand_1[31 : 16];
                multiplierCircuitInput2 <= operand_2[15 : 0];
                partialProduct2 <= multiplierCircuitResult;
                next_multiplication_stage <= 3'b011;
            end
            3'b011 : 
            begin
                multiplierCircuitInput1 <= operand_1[15 : 0];
                multiplierCircuitInput2 <= operand_2[31 : 16];
                partialProduct3 <= multiplierCircuitResult;
                next_multiplication_stage <= 3'b100;
            end
            3'b100 : 
            begin
                multiplierCircuitInput1 <= operand_1[31 : 16];
                multiplierCircuitInput2 <= operand_2[31 : 16];
                partialProduct4 <= multiplierCircuitResult;
                next_multiplication_stage <= 3'b101;
            end
            3'b101 :
            begin
                product <= partialProduct1 + (partialProduct2 << 16) + (partialProduct3 << 16) + (partialProduct4 << 32);
                next_multiplication_stage <= 3'b000;
                product_ready <= 1;
            end

            default: next_multiplication_stage <= 3'b000;
        endcase    
    end
```

Purpose: Controls the flow of the multiplication operation through different stages (3'b000 to 3'b101).
Behavior:
Initializes inputs and intermediate results (partialProduct1 to partialProduct4) at the beginning (3'b000).
Sequentially calculates partial products (partialProduct1 to partialProduct4) using the Multiplier module (multiplierCircuitResult).
Combines these partial products (product) to get the final result when all stages are completed (3'b101).
Signals product_ready when the multiplication operation is finished.


## outputs

![image](https://github.com/alihendi21/LUMOS/assets/170606621/b0611ca2-84be-4b5d-a585-debb8b02a382)

![image](https://github.com/alihendi21/LUMOS/assets/170606621/50d667e7-8c2b-4370-ae00-4f28b3d86183)

![image](https://github.com/alihendi21/LUMOS/assets/170606621/a777117e-6a09-404e-af7a-b3c52e3ef055)

![image](https://github.com/alihendi21/LUMOS/assets/170606621/b9c80479-4957-4cb4-9c1e-f2a3052abd78)

![image](https://github.com/alihendi21/LUMOS/assets/170606621/20e944b1-80b1-478e-abe8-71d906666466)

![image](https://github.com/alihendi21/LUMOS/assets/170606621/77f2a494-f061-462a-bd74-01336f2fa654)



