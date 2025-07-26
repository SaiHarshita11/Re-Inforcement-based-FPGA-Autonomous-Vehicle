**DESIGN SOURCE CODE: -**



//with clk code

`timescale 1ns / 1ps



module lfsr #(

&nbsp;   parameter WIDTH = 16,

&nbsp;   parameter POLYNOMIAL = 16'h8005

)(

&nbsp;   input clk,

&nbsp;   input rst,

&nbsp;   output reg \[WIDTH-1:0] q

);

&nbsp;   reg \[WIDTH-1:0] lfsr;



&nbsp;   always @(posedge clk or posedge rst) begin

&nbsp;       if (rst) begin

&nbsp;           lfsr <= {WIDTH{1'b1}};

&nbsp;       end else begin

&nbsp;           lfsr <= {lfsr\[WIDTH-2:0], lfsr\[WIDTH-1] ^ lfsr\[WIDTH-3] ^ lfsr\[WIDTH-5] ^ lfsr\[WIDTH-16]};

&nbsp;       end

&nbsp;   end



&nbsp;   always @(\*) begin

&nbsp;       q = lfsr;

&nbsp;   end

endmodule



//



module car(

&nbsp;   input clk,

&nbsp;   input rst,

&nbsp;   output reg \[0:6] seg,

&nbsp;   output reg \[7:0] DIGIT

);

integer k=0;



reg \[3:0] current\_x;

reg \[3:0] current\_y;

reg \[1:0] action;

reg signed\[15:0] reward;



reg \[2:0] digit\_select;     // 2 bit counter for selecting each of 4 digits

reg \[16:0] digit\_timer;     // counter for digit refresh

// Parameters for segment patterns

parameter ZERO  = 7'b000\_0001;  // 0

parameter ONE   = 7'b100\_1111;  // 1

parameter TWO   = 7'b001\_0010;  // 2 

parameter THREE = 7'b000\_0110;  // 3

parameter FOUR  = 7'b100\_1100;  // 4

parameter FIVE  = 7'b010\_0100;  // 5

parameter SIX   = 7'b010\_0000;  // 6

parameter SEVEN = 7'b000\_1111;  // 7

parameter EIGHT = 7'b000\_0000;  // 8

parameter NINE  = 7'b000\_0100;  // 9



reg \[3:0] start\_x = 4'b0000;

reg \[3:0] start\_y = 4'b0001;

reg \[3:0] end\_x = 4'b0011;

reg \[3:0] end\_y = 4'b1001;



parameter left = 2'b00;

parameter right = 2'b01;

parameter straight = 2'b10;

parameter back = 2'b11;



wire \[15:0] random\_value;

reg \[26:0] count;

reg seconds;

//

//

always @(posedge clk or posedge rst)

begin

&nbsp;   if(rst) begin

&nbsp;       count   <= 0;

&nbsp;       seconds <= 0;

&nbsp;   end else if (count == 27'd50\_000\_000) begin 

&nbsp;       count   <= 0;

&nbsp;       seconds <= ~seconds;

&nbsp;   end else begin

&nbsp;       count   <= count + 1'b1;    

&nbsp;   end 

end

//

//

lfsr #(

&nbsp;   .WIDTH(16),

&nbsp;   .POLYNOMIAL(16'h8005)

) lfsr\_inst (

&nbsp;   .clk(seconds),

&nbsp;   .rst(rst),

&nbsp;   .q(random\_value)

);

//

always @(posedge seconds or posedge rst)

&nbsp;begin

&nbsp;   if (rst) begin

&nbsp;       reward=+16'sb000000000000000;

&nbsp;       current\_x <= start\_x;

&nbsp;       current\_y <= start\_y;

&nbsp;       k = 0;

&nbsp;   end

&nbsp;

&nbsp;   else if (current\_x == end\_x \&\& current\_y == end\_y) begin

&nbsp;       k = 1;

&nbsp;       reward=reward+15;

&nbsp;       $finish;

&nbsp;   end

&nbsp;   else begin

&nbsp;       action = random\_value\[1:0]; 



&nbsp;       case (action)

&nbsp;           left: if (current\_x > 0 \&\& !(current\_x - 1 == 0 \&\& current\_y == 4) \&\& !(current\_x - 1 == 1 \&\& current\_y == 2) \&\& !(current\_x - 1 == 2 \&\& current\_y == 6) \&\& !(current\_x - 1 == 3 \&\& current\_y == 5))

&nbsp;            begin

&nbsp;               current\_x = current\_x - 1'b1;

&nbsp;               k = 0;

&nbsp;               reward=reward+5; //reward= +5

&nbsp;           end

&nbsp;           else if (current\_x<=0)

&nbsp;           reward=reward-7;  //reward= -7

&nbsp;          else if ((current\_x - 1 == 0 \&\& current\_y == 4) ||(current\_x - 1 == 1 \&\& current\_y == 2) || (current\_x - 1 == 2 \&\& current\_y == 6) || (current\_x - 1 == 3 \&\& current\_y == 5))

&nbsp;            reward=reward-10;  //reward=-10



&nbsp;           right: if (current\_x < 3 \&\& !(current\_x + 1 == 0 \&\& current\_y == 4) \&\& !(current\_x + 1 == 1 \&\& current\_y == 2) \&\& !(current\_x + 1 == 2 \&\& current\_y == 6) \&\& !(current\_x + 1 == 3 \&\& current\_y == 5)) begin

&nbsp;               current\_x = current\_x + 1'b1;

&nbsp;               k = 0;

&nbsp;               reward=reward+5; //reward= +5

&nbsp;           end

&nbsp;            else if (current\_x>=3)

&nbsp;           reward=reward-7;  //reward= -7

&nbsp;          else if((current\_x + 1 == 0 \&\& current\_y == 4) || (current\_x + 1 == 1 \&\& current\_y == 2) || (current\_x + 1 == 2 \&\& current\_y == 6) || !(current\_x + 1 == 3 \&\& current\_y == 5))

&nbsp;            reward=reward-10;  //reward=-10

&nbsp;           

&nbsp;         straight: if (current\_y < 9 \&\& !(current\_x == 0 \&\& current\_y + 1 == 4) \&\& !(current\_x == 1 \&\& current\_y + 1 == 2) \&\& !(current\_x == 2 \&\& current\_y + 1 == 6) \&\& !(current\_x == 3 \&\& current\_y + 1 == 5)) begin

&nbsp;               current\_y = current\_y + 1'b1;

&nbsp;               k = 0;

&nbsp;              reward=reward+5; //reward= +5

&nbsp;           end

&nbsp;            else if (current\_y>=9)

&nbsp;           reward=reward-7;  //reward= -7

&nbsp;          else if((current\_x == 0 || current\_y + 1 == 4) || (current\_x == 1 \&\& current\_y + 1 == 2) || (current\_x == 2 \&\& current\_y + 1 == 6) || (current\_x == 3 \&\& current\_y + 1 == 5))

&nbsp;            reward=reward-10;  //reward=-10

&nbsp;           back: if (current\_y>0 \&\& !(current\_x == 0 \&\& current\_y-1 == 4) \&\& !(current\_x== 1 \&\& current\_y-1 == 2) \&\& !(current\_x == 2 \&\& current\_y-1 == 6) \&\& !(current\_x == 3 \&\& current\_y-1 == 5)) 

&nbsp;           begin

&nbsp;               current\_y = current\_y - 1'b1;

&nbsp;               k = 0;

&nbsp;               reward=reward+5; //reward= +5

&nbsp;               end

&nbsp;                else if (current\_x<=3)

&nbsp;                reward=reward-7;  //reward= -7

&nbsp;                else if((current\_x == 0 \&\& current\_y-1 == 4) || (current\_x== 1 \&\& current\_y-1 == 2) || (current\_x == 2 \&\& current\_y-1 == 6) ||(current\_x == 3 \&\& current\_y-1 == 5)) 

&nbsp;                reward=reward-10;  //reward=-10

&nbsp;       endcase

&nbsp;   end

end



&nbsp;//

&nbsp;//

&nbsp;always @(posedge clk or posedge rst) begin

&nbsp;       if(rst) begin

&nbsp;           digit\_select <= 0;

&nbsp;           digit\_timer <= 0; 

&nbsp;       end

&nbsp;       else                                        // 1ms x 4 displays = 4ms refresh period

&nbsp;           if(digit\_timer == 99\_999) begin         // The period of 100MHz clock is 10ns (1/100,000,000 seconds)

&nbsp;               digit\_timer <= 0;                   // 10ns x 100,000 = 1ms

&nbsp;               digit\_select <=  digit\_select + 1;

&nbsp;           end

&nbsp;           else

&nbsp;               digit\_timer <=  digit\_timer + 1;

&nbsp;   end



&nbsp;   always @(digit\_select) begin

&nbsp;       case(digit\_select) 

&nbsp;           3'b000 : begin

&nbsp;                       DIGIT = 8'b11111110;

&nbsp;                   end

&nbsp;           3'b001 : begin

&nbsp;                       DIGIT = 8'b11111101;

&nbsp;                   end

&nbsp;          3'b010 : begin

&nbsp;                       DIGIT = 8'b11111011;

&nbsp;                   end

&nbsp;           3'b011 : begin

&nbsp;                       //DIGIT = 8'b11110111;

&nbsp;                       DIGIT = 8'b11111111;

&nbsp;                   end

&nbsp;           3'b100 : begin

&nbsp;                       //DIGIT = 8'b11101111;

&nbsp;                       DIGIT = 8'b11111111;

&nbsp;                   end

&nbsp;           3'b101 : begin

&nbsp;                       //DIGIT = 8'b11011111;

&nbsp;                       DIGIT = 8'b11111111;

&nbsp;                   end

&nbsp;           3'b110 : begin

&nbsp;                       //DIGIT = 8'b10111111;

&nbsp;                       DIGIT = 8'b11111111;

&nbsp;                   end

&nbsp;           3'b111 : begin

&nbsp;                       //DIGIT = 8'b01111111;

&nbsp;                       DIGIT = 8'b11111111;

&nbsp;                   end                                                                                

&nbsp;       endcase

&nbsp;   end

&nbsp;   always @\*

&nbsp;           case(digit\_select)

&nbsp;               3'b000 : begin

&nbsp;                           case(current\_y)

&nbsp;                               4'b0000: seg = ZERO;

&nbsp;                               4'b0001: seg = ONE;

&nbsp;                               4'b0010: seg = TWO;

&nbsp;                               4'b0011: seg = THREE;

&nbsp;                               4'b0100: seg = FOUR;

&nbsp;                               4'b0101: seg = FIVE;

&nbsp;                               4'b0110: seg = SIX;

&nbsp;                               4'b0111: seg = SEVEN;

&nbsp;                               4'b1000: seg = EIGHT;

&nbsp;                               4'b1001: seg = NINE;

&nbsp;                           endcase

&nbsp;                        end

&nbsp;               3'b001 : begin

&nbsp;                           case(current\_x)

&nbsp;                               4'b0000: seg = ZERO;

&nbsp;                               4'b0001: seg = ONE;

&nbsp;                               4'b0010: seg = TWO;

&nbsp;                               4'b0011: seg = THREE;

&nbsp;                               4'b0100: seg = FOUR;

&nbsp;                               4'b0101: seg = FIVE;

&nbsp;                               4'b0110: seg = SIX;

&nbsp;                               4'b0111: seg = SEVEN;

&nbsp;                               4'b1000: seg = EIGHT;

&nbsp;                               4'b1001: seg = NINE;

&nbsp;                           endcase

&nbsp;                        end

&nbsp;               3'b010 : begin

&nbsp;                           case(action)

&nbsp;                               4'b0000: seg = ZERO;

&nbsp;                               4'b0001: seg = ONE;

&nbsp;                               4'b0010: seg = TWO;

&nbsp;                               4'b0011: seg = THREE;

&nbsp;                               4'b0100: seg = FOUR;

&nbsp;                               4'b0101: seg = FIVE;

&nbsp;                               4'b0110: seg = SIX;

&nbsp;                               4'b0111: seg = SEVEN;

&nbsp;                               4'b1000: seg = EIGHT;

&nbsp;                               4'b1001: seg = NINE;

&nbsp;                           endcase

&nbsp;                        end

&nbsp;               3'b011 : seg = 7'b000\_0001;

&nbsp;               3'b100 : seg = 7'b000\_0001;

&nbsp;               3'b101 : seg = 7'b000\_0001;

&nbsp;               3'b110 : seg = 7'b000\_0001;

&nbsp;               3'b111 : seg = 7'b000\_0001;                                    

&nbsp;           endcase

//

//

endmodule 



