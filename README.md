## Iran Univeristy of Science and Technology
## Assignment : LUMOS RISC-V

- Name: seyed ali hendi
- Team Members: seyed ali hendi , amirhossein eslami , payam karimnejad
- Student ID: 99413262
- Date: 2024/05/25

## Overview


The provided Verilog code implements a Fixed-Point Unit (FPU) capable of performing various arithmetic operations such as addition, subtraction, multiplication, and square root calculations. The module is parameterized to handle fixed-point numbers of configurable width and fractional bits.

##1. Module Declaration and Parameters:
'''
module Fixed_Point_Unit 
(
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


'''


## Code Multiplier :

module Multi_part
(input wire [15 : 0] operand_1,input wire [15 : 0] operand_2,output reg [31 : 0] product);

    always @(*)
    begin
        product <= operand_1 * operand_2;
    end
endmodule

## outputs

![image](https://github.com/alihendi21/LUMOS/assets/170606621/b0611ca2-84be-4b5d-a585-debb8b02a382)

![image](https://github.com/alihendi21/LUMOS/assets/170606621/50d667e7-8c2b-4370-ae00-4f28b3d86183)

![image](https://github.com/alihendi21/LUMOS/assets/170606621/a777117e-6a09-404e-af7a-b3c52e3ef055)

![image](https://github.com/alihendi21/LUMOS/assets/170606621/b9c80479-4957-4cb4-9c1e-f2a3052abd78)

![image](https://github.com/alihendi21/LUMOS/assets/170606621/20e944b1-80b1-478e-abe8-71d906666466)

![image](https://github.com/alihendi21/LUMOS/assets/170606621/77f2a494-f061-462a-bd74-01336f2fa654)



