## Iran Univeristy of Science and Technology
## Assignment : LUMOS RISC-V

- Name: seyed ali hendi
- Team Members: seyed ali hendi , amirhossein eslami , payam karimnejad
- Student ID: 99413262
- Date: 2024/05/25

## Report

 Fixed Point Unit is reviewed, which is implemented in Verilog language. This unit is able to perform addition, subtraction, multiplication and square root operations. The purpose of this report is to fully explain each part of the code, review the various operations, and how to load and use values ​​in registers. Introduction of Fixed Point Unit, which is implemented in this project, is written in Verilog language and has various inputs that produce appropriate output depending on the type of operation. This unit includes various modules such as addition, subtraction, multiplication and square root calculation modules, which will be explained in detail below.

## Code Fixed_point_unit :

module Fixed_point_part 
#(parameter WIDTH = 32,parameter FBITS = 10)

(   input wire clk,
    input wire reset,
    input wire [WIDTH - 1 : 0] operand_1,
    input wire [WIDTH - 1 : 0] operand_2,
    input wire [ 1 : 0] operation,

    output reg [WIDTH - 1 : 0] result,
    output reg ready );


    always @(*)
    begin
        case (operation)
            `ADD_i1    : 
                        begin 
                            result = operand_1 + operand_2; 
                            ready = 1;
                        end
            `SUB_i2    : 
                        begin 
                            result = operand_1 - operand_2; 
                            ready = 1; 
                        end
            `MUL_i3    : 
                        begin 
                            result = product[WIDTH + FBITS - 1 : FBITS]; 
                            ready = product_ready; 
                        end
            `SQRT_i4   : 
                        begin 
                            result = root; 
                            ready = root_Ooutput; 
                        end
            default     : 
                        begin 
                            result = 'bz;
                            ready = 0; 
                        end
        endcase
    end

    always @(posedge reset)
    begin
        if (reset)  ready = 0;
        else        ready = 'bz;
    end
    


    // Square Root


    reg [WIDTH - 1 : 0] root;
    reg root_Ooutput;

    reg [1 : 0] SRS;
    reg [1 : 0] SRS_new;

    always @(posedge clk) 
    begin
        if (operation == `SQRT_i4) SRS <= SRS_new;
        else                        
        begin
            SRS <= 2'b00;
            root_Ooutput <= 0;
        end
    end 

    always @(*) 
    begin
        SRS_new <= 'bz;
        case (SRS)
            2'b00 : 
                    begin 
                        sqrt_start <= 0;
                        SRS_new <= 2'b01;
                    end
            2'b01 : 
                    begin 
                        sqrt_start <= 1;
                        SRS_new <= 2'b10;
                    end
            2'b10 : 
                    begin 
                        sqrt_start <= 0;
                        SRS_new <= 2'b10; 
                    end
        endcase    
    end

    reg sqrt_start;

    reg sqrt_busy;
    
    reg [WIDTH - 1 : 0] x, x_new;              
    reg [WIDTH - 1 : 0] q, q_new;              
    reg [WIDTH + 1 : 0] ac, ac_new; 

    reg [WIDTH + 1 : 0] test_res;               



    reg valid;

    localparam ITER = (WIDTH + FBITS) >> 1;     
    reg [4 : 0] i = 0;                              

    always @(*)
    begin
        test_res = ac - {q, 2'b01};

        if (test_res[WIDTH + 1] == 0) 
        begin
            {ac_new, x_new} = {test_res[WIDTH - 1 : 0], x, 2'b0};
            q_new = {q[WIDTH - 2 : 0], 1'b1};
        end 
        else 
        begin
            {ac_new, x_new} = {ac[WIDTH - 1 : 0], x, 2'b0};
            q_new = q << 1;
        end
    end
    
    always @(posedge clk) 
    begin
        if (sqrt_start)
        begin
            sqrt_busy <= 1;
            root_Ooutput <= 0;
            i <= 0;
            q <= 0;
            {ac, x} <= {{WIDTH{1'b0}}, operand_1, 2'b0};
        end

        else if (sqrt_busy)
        begin
            if (i == ITER-1) 
            begin  
                sqrt_busy <= 0;
                root_Ooutput <= 1;
                root <= q_new;
            end

            else 
            begin
                i <= i + 1;
                x <= x_new;
                ac <= ac_new;
                q <= q_new;
                root_Ooutput <= 0;
            end
        end
    end

  
    // Multiplier
   
    reg [64 - 1 : 0] product;
    reg product_ready;

    reg     [15 : 0] multi_Input1;
    reg     [15 : 0] multi_Input2;
    wire    [31 : 0] multi_Output;

    Multi_part multiplier_circuit
    (.operand_1(multi_Input1),.operand_2(multi_Input2),.product(multi_Output));

    reg [31 : 0] partialProduct1;
    reg [31 : 0] partialProduct2;
    reg [31 : 0] partialProduct3;
    reg [31 : 0] partialProduct4;
    reg [2 : 0] multiplication_stage;
    reg [2 : 0] multi_new;

    always @(posedge clk) 
    begin
        if (operation == `MUL_i3)  multiplication_stage <= multi_new;
        else                        multiplication_stage <= 'b0;
    end

    always @(*) 
    begin
        multi_new <= 'bz;
        case (multiplication_stage)

            3'b000 :
            begin
                product_ready <= 0;
                multi_Input1 <= 'bz;
                multi_Input2 <= 'bz;
                partialProduct1 <= 'bz;
                partialProduct2 <= 'bz;
                partialProduct3 <= 'bz;
                partialProduct4 <= 'bz;
                multi_new <= 3'b001;
            end 

            3'b001 : 
            begin
                multi_Input1 <= operand_1[15 : 0];
                multi_Input2 <= operand_2[15 : 0];
                partialProduct1 <= multi_Output;
                multi_new <= 3'b010;
            end

            3'b010 : 
            begin
                multi_Input1 <= operand_1[31 : 16];
                multi_Input2 <= operand_2[15 : 0];
                partialProduct2 <= multi_Output;
                multi_new <= 3'b011;
            end

            3'b011 : 
            begin
                multi_Input1 <= operand_1[15 : 0];
                multi_Input2 <= operand_2[31 : 16];
                partialProduct3 <= multi_Output;
                multi_new <= 3'b100;
            end

            3'b100 : 
            begin
                multi_Input1 <= operand_1[31 : 16];
                multi_Input2 <= operand_2[31 : 16];
                partialProduct4 <= multi_Output;
                multi_new <= 3'b101;
            end

            3'b101 :
            begin
                product <= partialProduct1 + (partialProduct2 << 16) + (partialProduct3 << 16) + (partialProduct4 << 32);
                multi_new <= 3'b000;
                product_ready <= 1;
            end

            default:
                multi_new <= 3'b000;
        endcase    
    end
endmodule


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



