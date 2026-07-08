# FSM
 - Melay 
 - Moore
 - Vending Machine
# Flipflops
 - D Flip flop
 - T Flip flop
 - JK Flip flop
# Memories
 - synchronous dual port 16x8
 - synchronous dual port 8x16
 - Asunchronous dual port 
 - Asunchronous single port
## Melay
### [RTL]
```bas
//Melay overlapping sequence detector 1101, LSB first(1011)
module FSM(clk,reset,in,out);
parameter s0=2'b00,
          s1=2'b01,
			 s2=2'b10,
			 s3=2'b11;
input clk,reset,in;
output reg out;
reg [1:0]pr,nx;
always@(posedge clk or posedge reset)
begin
if(reset)
 pr<=s0;
else
 pr<=nx;
end
always@(*)
begin
case(pr)
s0: if(in==1)
     nx=s1;
	  else
	  nx=s0;
s1: if(in==0)
     nx=s2;
	  else
	  nx=s1;
s2: if(in==1)
     nx=s3;
	  else
	  nx=s0;
s3: if(in==1)
     nx=s1;
	  else
	  nx=s2;
endcase
end
always@(posedge clk)
begin
  out<=0;
  case(pr)
	  s3: if(in==1)
	    out<=1'b1;  
 endcase
end
endmodule
```
### [Test bench]
```bash
module FSM_tb;
	reg clk;
	reg reset;
	reg in;
	wire out;
   parameter CYCLE=10;
	FSM fsm(
		.clk(clk), 
		.reset(reset), 
		.in(in), 
		.out(out)
	);
	always begin
	 #(CYCLE/2);
    clk=1'b0;
	  #(CYCLE/2);
	 clk=~clk;
	//Step1 : Generate clock, using parameter "CYCLE"  
    end
     task initialize();
	    begin
        in=0;   
       end	
     endtask		 		 
   //Delay task
   task delay(input integer i);
      begin
	 #i;
      end
   endtask
  task RESET;
  begin 
   delay(5);
	reset=1'b1;
	delay(10);
	reset=1'b0;
	end
	endtask
      task stimulus(input integer b);
		begin
		@(negedge clk)
		in=b;
		end
		endtask		 
   //Process to monitor the changes in the variables
   initial 
      $monitor("Reset=%b, pr=%b, clk=%b, in=%b,out=%b ",
	       reset,fsm.pr,clk,in,out);
always@(fsm.pr or out)
      begin
	 if(fsm.pr==2'b11 && out==1)
	    $display("Correct output at state %b", fsm.pr);
      end
	initial begin
         initialize;
	 RESET;
	 stimulus(0);
	 stimulus(1);
	 stimulus(0);
	 stimulus(1);
	 stimulus(0);
	 stimulus(1);
	 stimulus(1);
	 RESET;
	 stimulus(1);
	 stimulus(0);
	 stimulus(1);
	 stimulus(1);
	 delay(10);    
	 $finish;
      end	
endmodule
```
## Moore
### [RTL]
```bash
//Moore overlapping 1011
module seq_det(seq_in,clock,reset,det_o);
								 
  parameter IDLE=2'b00,
            STATE1=2'b01,
            STATE2=2'b10,
            STATE3=2'b11;	
   input seq_in,clock,reset;
   output det_o;	
   //Internal registers
   reg [1:0]state,next_state;
  always@(posedge clock or posedge reset)
  begin
     if(reset)
        state<=IDLE;
     else
        state<=next_state;//Step3 : Write down the sequential logic for present state with active high asychronous reset
	end
   //Understand the combinational logic for next state
   always@(state,seq_in)
      begin
	 case(state)
	    IDLE   : 
                      if(seq_in==1) 
		         next_state=STATE1;
	              else
	                 next_state=IDLE;
	    STATE1 : 
                      if(seq_in==0)
	                 next_state=STATE2;
	              else
	                 next_state=STATE1;
	    STATE2 :
                      if(seq_in==1)
	                 next_state=STATE3;
	              else 
	                 next_state=IDLE;
	    STATE3 : 
                      if(seq_in==1)
	                 next_state=STATE1;
	              else 
	                 next_state=STATE2;
	    default: 
                      next_state=IDLE;
	 endcase
      end
   assign det_o=(state==STATE3)?1'b1:1'b0; //Step4 : Write down the logic for Moore output det_o
endmodule
```
### [Test bench]
```bash
module seq_det_tb();
		
   //Testbench global variables
   reg  din,clock,reset;
   wire dout;
  //Parameter constant for CYCLE
   parameter CYCLE = 10;
   //DUT Instantiation
   seq_det SQD(.seq_in(din),
	       .clock(clock),
	       .reset(reset),
	       .det_o(dout));
   always begin
	 #(CYCLE/2);
    clock=1'b0;
	  #(CYCLE/2);
	 clock=~clock;
	//Step1 : Generate clock, using parameter "CYCLE"  
    end
     task initialize();
	    begin
        din=0;   
       end	
     endtask		  /*Step2 : Write a task named "initialize" to initialize 
      		  the input din of sequence detector*/		 
   //Delay task
   task delay(input integer i);
      begin
	 #i;
      end
   endtask
  task RESET;
  begin 
   delay(5);
	reset=1'b1;
	delay(10);
	reset=1'b0;
	end
	endtask
  /*Step3 : Write a task named "RESET" to reset the design,
            use delay task for adding delays*/
      task stimulus(input integer b);
		begin
		@(negedge clock)
		din=b;
		end
		endtask
   /*Step4 : Write a task named "stimulus" which provides input to
            design on negedge of clock*/			 
   //Process to monitor the changes in the variables
   initial 
      $monitor("Reset=%b, state=%b, Din=%b, Output Dout=%b ",
	       reset,SQD.state,din,dout);		
   /*Process to display a string after the sequence is detected and dout is asserted.
   SQD.state is used here as a path hierarchy where SQD is the instance name acting
   like a handle to access the internal register "state" */
   always@(SQD.state or dout)
      begin
	 if(SQD.state==2'b11 && dout==1)
	    $display("Correct output at state %b", SQD.state);
      end	
   /*Process to generate stimulus by calling the tasks and 
   passing the sequence in an overlapping mode*/		
   initial
      begin
         initialize;
	 RESET;
	 stimulus(0);
	 stimulus(1);
	 stimulus(0);
	 stimulus(1);
	 stimulus(0);
	 stimulus(1);
	 stimulus(1);
	 RESET;
	 stimulus(1);
	 stimulus(0);
	 stimulus(1);
	 stimulus(1);
	 delay(10);    
	 $finish;
      end	
endmodule     
```
## Vending Machine
### [RTL]
```bash
//Melay model
module vending(clk,reset,in,P,R);
parameter s0=2'b00,
          s1=2'b01,
			 s2=2'b10;
input clk,reset;
input [1:0]in;
output reg P,R;
reg [1:0]pr,nx;
always@(posedge clk or posedge reset)
begin
if(reset)
 pr<=s0;
else
 pr<=nx;
end
always@(*)
begin
case(pr)
s0: if(in==1)
     nx=s1;
	  else if(in==2)
	  nx=s2;
s1: if(in==1)
     nx=s2;
	  else if(in==2)
	  nx=s0;
	  else
	  nx=s1;
s2: if(in==1||in==2)
     nx=s0;
	  else
	  nx=s2;
	default: nx = s0;  
endcase
end
always@(*)
begin
  P<=0;
  R<=0;
 case(pr)
  s1: if(in==2)
      P<=1;
  s2: if(in==1)
      P<=1;
  else if(in==2)
   begin
      P<=1;
		R<=1;
		end
endcase
end
endmodule
//Moore model
module vending(clk,reset,in,P,R);
parameter s0=4'b0000,
          s1=4'b0100,
			 s2=4'b1000,
			 s3=4'b0010,
			 s4=4'b0011;
input clk,reset;
input [1:0]in;
output P,R;
reg [3:0]pr,nx;
always@(posedge clk or posedge reset)
begin
if(reset)
 pr<=s0;
else
 pr<=nx;
end
always@(*)
begin
case(pr)
s0: if(in==1)
     nx=s1;
	  else if(in==2)
	  nx=s2;
s1: if(in==1)
     nx=s2;
	  else if(in==2)
	  nx=s3;
	  else
	  nx=s0;
s2: if(in==1)
     nx=s3;
	  else if(in==2)
	  nx=s4;
s3: nx=s0;
s4: nx=s0;
	default: nx = s0;  
endcase
end
assign {P,R}=pr[1:0];
endmodule
```
### [Test bench]
```bash
module vending_tb;
	// Inputs
	reg clk;
	reg reset;
	reg [1:0] in;
	wire P;
	wire R;
   parameter CYCLE=10;
	// Instantiate the Unit Under Test (UUT)
	vending uut (
		.clk(clk), 
		.reset(reset), 
		.in(in), 
		.P(P), 
		.R(R)
	);
      always begin
	 #(CYCLE/2);
    clk=1'b0;
	  #(CYCLE/2);
	 clk=1'b1;
	//Step1 : Generate clock, using parameter "CYCLE"  
    end
     task initialize();
	    begin
        in=2'b00;   
       end	
     endtask		 		 
   //Delay task
   task delay(input integer i);
      begin
	 #i;
      end
   endtask
  task RESET;
  begin 
   delay(5);
	reset=1'b1;
	delay(10);
	reset=1'b0;
	end
	endtask
      task stimulus(input [1:0]b);
		begin
		@(negedge clk)
		in=b;
		end
		endtask		
		// Initialize Inputs
		initial begin
      $monitor("Reset=%b, pr=%b, clk=%b, in=%b,P=%b R=%b",
	       reset,uut.pr,clk,in,P,R);
      end
 //for moore
always@(uut.pr or P or R)
      begin
	 if(uut.pr==4'b0010 && P==1)
	    $display("Correct output at state %b", uut.pr);
		 	else if(uut.pr==4'b0011 && P==1)
	    $display("Correct output at state %b", uut.pr);
    	 if(uut.pr==4'b0011 && P==1 && R==1)
	    $display("Correct output at state %b", uut.pr);
      end
		/*//for melay
		always@(uut.pr or P or R)
      begin
	 if(uut.pr==2'b01 && P==1)
	    $display("Correct output at state %b", uut.pr);
		 	else if(uut.pr==2'10 && P==1)
	    $display("Correct output at state %b", uut.pr);
    	 if(uut.pr==2'b10 && P==1 && R==1)
	    $display("Correct output at state %b", uut.pr);
      end*/
	initial begin
		// Initialize Inputs
         initialize;
	 RESET;
	 stimulus(2'b10);
	 stimulus(2'b01);
	 stimulus(2'b10);
	 stimulus(2'b10);
	 RESET;
	 stimulus(2'b00);
	 stimulus(2'b01);
	 stimulus(2'b11);
	 stimulus(2'b00);
	 delay(10); 
	 #100 $finish;
      end	
endmodule
```
## D Flip flop
### [RTL]
```bash
module dff(clock,reset,d_in,Q_out,Qb_out);
  input clock,reset,d_in;
	output reg Q_out;
	output Qb_out;
   always@(posedge clock)
      begin
	 if(reset)
	    Q_out <= 1'b0;
	 else
	    Q_out <= d_in;
      end		
	 assign Qb_out=~(Q_out); 
endmodule          
```
### [Test bench]
```bash
module dff_tb();
   reg clk,reset,d;
   wire q,qb;
   parameter CYCLE = 10;//Step1 : Define a parameter with name "CYCLE" which is equal to 10  
   dff DUT(.clock(clk),.reset(reset),.d_in(d),.Q_out(q),.Qb_out(qb));
   always
      begin
	 #(CYCLE/2);
	 clk = 1'b0;
	 #(CYCLE/2);
	 clk = 1'b1;
      end
   //Reset task
   task rst_dut();
      begin
	 @(negedge clk);
	 reset=1'b1;
	 @(negedge clk);
	 reset=1'b0;
      end
   endtask
   //Data task
   task din(input i);
      begin
         @(negedge clk);
         d=i;
      end
   endtask
   initial 
      begin
         rst_dut;
         din(0);
         din(1);
         din(0);
         din(1);
         din(1);
         rst_dut;
         din(0);
         din(1);
         #10; 
      end
		initial begin
      #1000 $finish;
		end
		initial begin
   $monitor("clk=%b,reset=%b,d=%b,q=%b,qb=%b",clk,reset,d,q,qb);
      end
endmodule
```
## T Flip flop
### [RTL]
```bash
module tff(clock,reset,t,q,qb);
input clock,reset,t;
output qb,q;
wire w1;
xor x1(w1,t,q);
d_ff d1(.clock(clock),.reset(reset),.d_in(w1),.Q_out(q),.Qb_out(qb));
//assign qb=~(q); 
endmodule
module d_ff(clock,
	   reset,
	   d_in,
	   Q_out,
	   Qb_out);

   input clock,reset,d_in;
	output reg Q_out;
	output Qb_out;//Step1 : Declare Port Directions

   /*Understand the Behaviour of D flip-flop &
   check the coding style of synchronous reset*/

   always@(posedge clock) //or posedge reset)
      begin
	 if(reset)
	    Q_out <= 1'b0;
	 else
	    Q_out <= d_in;
      end

   //Step2 : Write the logic for Qbar				
	 assign Qb_out=~(Q_out); 
endmodule          
```
### [Test bench]
```bash
module tff_tb;
	// Inputs
	reg clock;
	reg reset;
	reg t;
	// Outputs
	wire q;
	wire qb;
	// Instantiate the Unit Under Test (UUT)
	tff uut (
		.clock(clock), 
		.reset(reset), 
		.t(t), 
		.q(q), 
		.qb(qb)
	);
	always
	begin
	 clock=1'b0;
	 #10;
	 clock=1'b1;
	 #10;
	 end
	 task re_set;
	 begin
	 @(negedge clock);
	  reset=1'b1;
	  #5;
	 @(negedge clock);
	  reset=1'b0;
	  end
	  endtask
	  task in_put(input in);
	  begin
	  @(negedge clock);
	   t=in;
		end
		endtask
		initial begin
		clock = 0;
		reset = 0;
		t = 0;
		#100;
		   re_set;
			in_put(0);
			in_put(1);
			in_put(0);
			in_put(0);
			re_set;
			in_put(0);
			in_put(0);
			in_put(1);
			#10;
	end
    initial begin
      #1000$finish;
		end
		initial begin
   $monitor("clk=%b,reset=%b,t=%b,q=%b,qb=%b",clock,reset,t,q,qb);
      end  
endmodule
```
## JK Flip flop
### [RTL]
```bash
module jkff(clk,reset,j,k,q,qb);
input j,k,reset,clk;
output reg q;
output qb;
parameter HOLD = 2'b00,
          RESET = 2'b01,
          SET   = 2'b10,
		  TOGGLE = 2'b11;
always @(posedge clk)
   begin
     if(reset)
	    begin
		   q<=0;
		 end
	  else 
	      begin
			case({j,k})
			 HOLD : q<=q;
			 RESET : q<=0;
			 SET : q<=1;
			 TOGGLE : q<=~q;
			 endcase
			 end
	end
	assign qb=~q;
endmodule
//behavioural model
always @(posedge clk)
   begin
     if(reset)
	    begin
		   q<=0;
		 end
	  else 
	      begin
			case({j,k})
			 2'b00 : q<=q; 2'b01 : q<=0;
			 2'b10 : q<=1;2'b11 : q<=~q;
			 endcase
			 end
	end
	assign qb=~q;
endmodule
```
### [Test bench]
```bash
module jkff_tb;
	// Inputs
	reg clk;
	reg reset;
	reg j;
	reg k;
	// Outputs
	wire q;
	wire qb;
   integer o;
	// Instantiate the Unit Under Test (UUT)
	jkff uut (.clk(clk), .reset(reset), .j(j), .k(k), .q(q), .qb(qb));
	always
	begin
	 clk=1'b0;
	 #10;
	 clk=1'b1;
	 #10;
	 end
   task re_set();
      begin
	 @(negedge clk);
	 reset=1'b1;
	 @(negedge clk);
	 reset=1'b0;
      end
   endtask
   task in(input m, input n);
      begin
         @(negedge clk);
         j=m;
			k=n;
      end
   endtask
	initial begin
		clk = 0;
		reset = 0;
		j = 0;
		k = 0;
		#100;
      re_set;
		in(0,1);in(1,0);
		in(1,1);in(0,1);
		in(1,1);in(0,0);
		in(1,1);in(0,1);
		in(1,1);in(0,0);
		in(1,1);in(1,0);
		in(1,1);in(0,0);
		#10;
		re_set;
      for(o=0;o<4;o=0+1)
		begin 
		 {j,k}=o;
		 #10;
		end
	end
    initial begin
      #500 $finish;
		end
		initial begin
   $monitor("clk=%b,reset=%b,j=%b,k=%b,q=%b,qb=%b",clk,reset,j,k,q,qb);
      end
endmodule
```
## synchronous dual port 16x8 
### [RTL]
```bash
module ram16_8(clk,din,dout,reset,we,re,wa,ra);
parameter WIDTH=8,
          DEPTH=16,
			 add=4;
input re,we,clk,reset;
input [add-1:0]wa,ra;
input [WIDTH-1:0]din;
output reg [WIDTH-1:0]dout;
reg [WIDTH-1:0]mem[DEPTH-1:0];
integer i;
always@(posedge clk or posedge reset)
begin
if(reset)
	begin
for(i=0;i<DEPTH;i=i+1)
mem[i]<=0;			
end
else
begin
if(we==1)
mem[wa]<=din;
if(re==1)
dout<=mem[ra];
end
end
endmodule
```
### [Test bench]
```bash
module ram16_8_tb;
	reg clk;
	reg [7:0] din;
	reg reset;
	reg we;
	reg re;
	reg [3:0] wa;
	reg [3:0] ra;
	wire [7:0] dout;
	// Instantiate the Unit Under Test (UUT)
	ram16_8 uut (.clk(clk), .din(din), .dout(dout), .reset(reset), .we(we), .re(re), .wa(wa), .ra(ra));
	    always
		  begin
		  clk=0;
		  #5;
		  clk=1;
		  #5;
		  end
		  task re_set();
		  begin
		  @(negedge clk)
		  reset=1'b1;
		  @(negedge clk)
		  reset=1'b0;
		  end
		  endtask
		  task write(input [3:0]a,input m,input [7:0]l);
		  begin
		  @(negedge clk)
		  din=l;
		  wa=a;
		  we=m;
		  end
		  endtask
		  task read(input [3:0]b,input n);
		  begin
		  @(negedge clk)
		  ra=b;
		  re=n;
		  end
		  endtask
	initial begin
		clk = 0;
		din = 0;
		reset = 0;
		we = 0;
		re = 0;
		wa = 0;
		ra = 0;
		#100;
		re_set;
		write(4'b1000,1,8'b11110000);
		#10;
		write(4'b0001,1,8'b00001111);
		#10;
		write(4'b0000,0,8'b11111111);
		#10;
		read(4'b1000,1'b1);
		#10;
		write(4'b1111,1,8'b11111111);
		read(4'b1111,1'b1);
      #10;
		read(4'b0001,1'b1);
      #10;
		read(4'b0000,1'b0);
      #10;
	end
      initial begin
        $monitor("clk=%b reset=%b we=%b re=%b wa=%b ra=%b din=%b dout=%b",clk,reset,we,re,wa,ra,din,dout);
      end
      initial begin
       #1000 $finish;
      end		 
endmodule
```
## synchronous dual port 8x16
### [RTL]
```bash
module ram8_16(clk,din,dout,reset,we,re,wa,ra);
parameter WIDTH=16,
          DEPTH=8,
			 add=3;
input re,we,clk,reset;
input [add-1:0]wa,ra;
input [WIDTH-1:0]din;
output reg [WIDTH-1:0]dout;
reg [WIDTH-1:0]mem[DEPTH-1:0];
always@(posedge clk or posedge reset)
begin
if(reset)
begin
for(i=0;i<DEPTH;i=i+1)
mem[i]<=0;
end
else
begin
if(we==1)
mem[wa]<=din;
if(re==1)
dout<=mem[ra];
end
end
endmodule
```
### [Test bench]
```bash
module ram8_16_tb;

	// Inputs
	reg clk;
	reg [15:0] din;
	reg reset;
	reg we;
	reg re;
	reg [2:0] wa;
	reg [2:0] ra;

	// Outputs
	wire [15:0] dout;

	// Instantiate the Unit Under Test (UUT)
	ram8_16 uut (
		.clk(clk), 
		.din(din), 
		.dout(dout), 
		.reset(reset), 
		.we(we), 
		.re(re), 
		.wa(wa), 
		.ra(ra)
	);
    always
		  begin
		  clk=0;
		  #5;
		  clk=1;
		  #5;
		  end
		  task re_set();
		  begin
		  @(negedge clk)
		  reset=1'b1;
		  @(negedge clk)
		  reset=1'b0;
		  end
		  endtask
		  task write(input[2:0]m,input n,[15:0]p);
		  begin
		  din=p;
		  wa=m;
		  we=n;
		  end 
		  endtask
		  task read([2:0]k,input l);
		  begin
		  ra=k;
		  re=l;
		  end 
		  endtask
	initial begin
		// Initialize Inputs
		clk = 0;
		din = 0;
		reset = 0;
		we = 0;
		re = 0;
		wa = 0;
		ra = 0;
		// Wait 100 ns for global reset to finish
		#100;
       write(001,1,16'hA);
		 #10;
		 write(111,1,16'h8);
		 #10;
		 write(101,0,16'h1);
		 #10;
		 write(011,1,16'h6);
		 #10;
		 read(001,1);
		 #10;
		 read(111,1);
		 #10;
		 read(101,0);
		 #10;
		 read(011,1);
		 #10;
		// Add stimulus here

	end
      initial begin
        $monitor("clk=%b reset=%b we=%b re=%b wa=%b ra=%b din=%h dout=%h",clk,reset,we,re,wa,ra,din,dout);
      end
      initial begin
       #200 $finish;
      end	
endmodule
```
## Asynchronous dual port 
### [RTL]
```bash
module ram16_8(wr_clk,rd_clk,din,dout,reset,we,re,wa,ra);
parameter WIDTH=8,
          DEPTH=16,
			 add=4;
input re,we,wr_clk,rd_clk,reset;
input [add-1:0]wa,ra;
input [WIDTH-1:0]din;
output reg [WIDTH-1:0]dout;
reg [WIDTH-1:0]mem[DEPTH-1:0];
integer i;
always@(posedge wr_clk or posedge reset)
begin
if(reset)
 begin
 for(i=0;i<DEPTH;i=i+1)
 mem[i]<=0;
     end
else
begin
if(we)
mem[wa]<=din; //if single poet dount<=mem[wa]
    end
end
always@(posedge rd_clk or posedge reset) //depends on only read
begin
if(reset)
begin
dout<=0;
end
else
begin
if(re)
dout<=mem[ra];
end
end
endmodule
```
### [Test bench]
```bash
module ram16_8_tb;

	// Inputs
	reg wr_clk;
	reg rd_clk;
	reg [7:0] din;
	reg reset;
	reg we;
	reg re;
	reg [3:0] wa;
	reg [3:0] ra;
  
	// Outputs
	wire [7:0] dout;	// Instantiate the Unit Under Test (UUT)
	ram16_8 uut (
	   .wr_clk(wr_clk),
		.rd_clk(rd_clk), 
		.din(din), 
		.dout(dout), 
		.reset(reset), 
		.we(we), 
		.re(re), 
		.wa(wa), 
		.ra(ra)
	);
	    always
		  begin
		  wr_clk=0;
		  #5;
		  wr_clk=1;
		  #5;
		  end
		  always
		  begin
		  rd_clk=0;
		  #5;
		  rd_clk=1;
		  #5;
		  end
		  
		  task re_set;
			begin
				@(negedge wr_clk or negedge rd_clk)
					reset=1'b1;
				@(negedge wr_clk or negedge rd_clk)
					reset=1'b0;
		  end
		  endtask
		  task write(input [3:0]a,input m,input [7:0]l);
		  begin
		  @(negedge wr_clk)
		  din=l;
		  wa=a;
		  we=m;
		  end
		  endtask
		  task read(input [3:0]b,input n);
		  begin
		  @(negedge rd_clk)
		  ra=b;
		  re=n;
		  end
		  endtask
		  
		  task wr_rd(input i,input y,input[3:0]q,input[7:0]x,input[3:0]s);
			begin
				@(negedge rd_clk or negedge wr_clk)
				we=i;
				re=y;
				wa=q;
				ra=s;
				din=x;
				#10;
			end
		endtask

	initial begin
		// Initialize Inputs
		wr_clk = 0;
		rd_clk=0;
		din = 0;
		reset = 1;
		we = 0;
		re = 0;
		wa = 0;
		ra = 0;
		#100;
		re_set;
		write(4'b1000,1,8'b11110000);
		#10;
		write(4'b0001,1,8'b00001111);
		#10;
		write(4'b0010,0,8'b11111111);
		#10;
		write(4'b1101,1,8'b11111100);
		#10;
		read(4'b1000,1'b1);
		#10;
		read(4'b0001,1'b1);
      #10;
		read(4'b0010,1'b0);
      #10;
		read(4'b1101,1'b1);
      #10;
		wr_rd(1,1,4'b1111,8'b00001001,4'b1111);
		#10;
	end
      initial begin
        $monitor("wr_clk=%b rd_clk=%b reset=%b we=%b re=%b wa=%b ra=%b din=%b dout=%b",wr_clk,rd_clk,reset,we,re,wa,ra,din,dout);
      end
      initial begin
       #800 $finish;
      end		 
endmodule
```
## Asunchronous single port
### [RTL]
```bash
module asy_single_port(input wr_in,
	input rd_in,
	input [3:0]address,
	input [7:0]data_in,output [7:0]data_out);
	 
	 reg [7:0]mem[15:0];
	 always@(wr_in,rd_in,address,data_in) // not depends on clk
     begin
     if(wr_in&&!rd_in)
     mem[address]=data_in;
     end	
	assign data_out=(rd_in&&!wr_in)? mem[address]:8'hzz;
endmodule
```
### [Test bench]
```bash
module asy_single_port_tb;

	// Inputs
	reg wr_in;
	reg rd_in;
	reg [3:0] address;
   reg [7:0] data_in;
	// Bidirs
	wire [7:0] data_out;

	// Instantiate the Unit Under Test (UUT)
	asy_single_port uut (
		.wr_in(wr_in), 
		.rd_in(rd_in), 
		.address(address), 
		.data_in(data_in),
		.data_out(data_out)
	); 
	task write(input [3:0]addr,input [7:0]data);
	begin 
	 wr_in=1;
	 rd_in=0;
	 address=addr;
	 data_in=data;
	 #10;
	 wr_in=0;
	end
	endtask
	
	task read(input[3:0]b);
	begin 
	 rd_in=1;
	 wr_in=0;
	 address=b;
	 #10;
	 rd_in=0;
	end
	endtask

	initial begin
		// Initialize Inputs
		wr_in = 0;
		rd_in = 0;
		address = 0;
		#100;
        write(4'b0001,8'b10101010);
		  write(4'b0010,8'b11110000);
		  write(4'b0011,8'b00001111);
		  #10;
		  read(4'b0001);
		  read(4'b0010);
		  read(4'b0011);
		  //read(0,4'b0010);
		  #10;
		// Add stimulus here

	end
     initial begin
        $monitor(" wr_in=%b, rd_in=%b,address=%b,data_in%b,data_out=%b ",wr_in,rd_in,address,data_in,data_out);
      end
      initial begin
       #1000 $finish;
      end	 
endmodule
```
