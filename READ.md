***Goofy-Gadgets***
# sequential circuits
### - latch
  - SR latch
### - Shift Registers
  - siso
  - sipo
  - piso
  - pipo
  - Universal shift
### - Counters
  - sync up(4-bit)
  - 3_bit asyn up counter posedge
  - 3_bit asyn down counter posedge
  - mod12_upcounter
  - bidirectional up-down
  - Ring counter
  - Johnson counter
  - Gray code counter
## SR latch-[RTL]
```bash
module SR_latch(s,r,q,qb);
input s,r;
output q,qb;
nor n1(q,r,qb);
nor n2(qb,s,q);
endmodule
```
## [Test bench]
```bash
module SR_latch_tb;
	reg s;
	reg r;
	wire q;
	wire qb;
	// Instantiate the Unit Under Test (UUT)
	SR_latch uut (.s(s), .r(r), .q(q), .qb(qb));
	initial begin
		// Initialize Inputs
		s = 0;
		r = 0;
		// Wait 100 ns for global reset to finish
		#100;
		  s=0;r=1;#10;
          s=0;r=0;#10;
		  s=1;r=0;#10;
		  s=1;r=1;#10;
		  s=0;r=1;#10;
          s=0; r=0;#10;
		  s=1; r=0;#10;
		  s=1; r=1;#10;
	end
     initial begin
      #1000$finish;
		end
		initial begin
   $monitor("s=%b,r=%b,q=%b,qb=%b",s,r,q,qb);
      end   
endmodule
```
# Siso
## [RTL]
```bash
module sis0(clk,reset,data,out);
input clk,reset;
input data;
output out;
reg [3:0]s_r;
always@(posedge clk)
begin
 if(reset)
	 s_r<=4'b0000;
 else
	 s_r<={s_r[2:0],data};
	 end
assign out=s_r[3];
endmodule
```
## [Test bench]
```bash
module siso_tb;
	reg clk;
	reg reset;
	reg data;
	wire [3:0] s_r;
	wire out;
	sis0 uut (.clk(clk), .reset(reset), .data(data), .out(out));
	 always #5 clk = ~clk;
	initial begin
		clk = 0;
    	reset = 1;
		 data = 0;
		  #10;
          reset=0;
		  data=1;
		  #10;
		  data=0;
		  #10;
		  data=1;
		  #10;
		  data=0;
		  #10;
	end
       initial begin
      #120 $finish;
		end
		initial begin
   $monitor("clk=%b,reset=%b, data=%b,out=%b",clk,reset,data,out);
      end
endmodule
```
# sipo
## [RTL]
```bash
module sipo(clk,reset,data,prll_out);
input clk,reset;
input data;
output [3:0]prll_out;
reg [3:0]s_r;
always@(posedge clk)
begin
 if(reset)
	 s_r<=4'b0000;
 else
	 s_r<={s_r[2:0],data};
end
assign prll_out=s_r;
endmodule
```
## [Test bench]
```bash
module sipo_tb;
	reg clk;
	reg reset;
	reg data;
	wire [3:0] prll_out;
	sipo uut (
		.clk(clk), 
		.reset(reset), 
		.data(data), 
		.prll_out(prll_out)
	);
     always #5 clk=~clk;
	initial begin
		// Initialize Inputs
		clk = 0;
		reset = 1;
		data = 0;
		#10;
        reset=0;
		  data=1;
		  #10;
		  data=0;
		  #10;
		  data=1;
		  #10;
		  data=1;
		  #10;
	end
       initial begin
      #120 $finish;
		end
		initial begin
   $monitor("clk=%b,reset=%b, data=%b,prll_out=%b",clk,reset,data,prll_out);
	//Step3 : Use $monitor to display the various inputs and outputs
      end
endmodule
```
# piso
## [RTL]
```bash
module piso(clk,reset,load,serial_out,prll_in);
input clk,reset;
input load;
output [3:0]prll_in;
output serial_out;
reg [3:0]s_r;
always@(posedge clk or posedge reset)
begin
 if(reset)
	 s_r<=4'b0000;
 else if(load)
    s_r<=prll_in;
	 else
	 s_r<={s_r[2:0],1'b0};
end
assign serial_out=s_r[3];
endmodule
```
## [Test bench]
```bash
module piso_tb;
	reg clk;
	reg reset;
	reg load;
	wire serial_out;
	reg [3:0] prll_in;
	piso uut (.clk(clk), .reset(reset), .load(load), .serial_out(serial_out), .prll_in(prll_in));
      always#5 clk=~clk;
	 initial begin
		clk = 0;
		reset = 1;
		load = 0;
      prll_in=4'b0000;
		#100;
		  reset=0;
        load=1;
		  prll_in=4'b0010;
		  #10;
		  load=0;
		  #40;
		  reset=0;
          load=1;
		  prll_in=4'b1010;
		  #10;
		  load=0;
		  #40;
	end
       initial begin
      #500 $finish;
		end
		initial begin
   $monitor("clk=%b,reset=%b, prll_in=%b,serial_in=%b",clk,reset,serial_out,prll_in);
      end
endmodule
```
# pipo
## [RTL]
```bash
module pipo(clk,reset,prll_out,prll_in);
input clk,reset;
input[3:0]prll_in;
output reg[3:0]prll_out;
always@(posedge clk or posedge reset)
begin
 if(reset)
	 prll_out<=4'b0000;
 else 
    prll_out<=prll_in;
end
endmodule
```
## [Test bench]
```bash
module pipo_tb;

	// Inputs
	reg clk;
	reg reset;
	reg [3:0] prll_in;

	// Outputs
	wire [3:0] prll_out;

	// Instantiate the Unit Under Test (UUT)
	pipo uut (
		.clk(clk), 
		.reset(reset), 
		.prll_out(prll_out), 
		.prll_in(prll_in)
	);
	always #5 clk=~clk;

	initial begin
		// Initialize Inputs
		clk = 0;
		reset = 1;
		prll_in = 4'b0000;

		// Wait 100 ns for global reset to finish
		#100;
		reset=0;
		prll_in=4'b1010;
		#10; // Add stimulus here
      prll_in=4'b1110;
		#10;
		prll_in=4'b0011;
		#10;
		prll_in=4'b0001;
		#10;
	end
       initial begin
      #200 $finish;
		end
		initial begin
   $monitor("clk=%b,reset=%b, prll_in=%b,prll_out=%b",clk,reset,prll_in,prll_out);
	//Step3 : Use $monitor to display the various inputs and outputs
      end
endmodule
```
# Universal shift
## [RTL]
```bash
module Universal_shift((clk,reset,load,prl_in,prl_out);
input clk,reset;
input [1:0]load;
input [3:0]prl_in;
output reg[3:0]prl_out;
always@(posedge clk or posedge reset)
begin
  if(reset)
  begin
     prl_out<=4'b0000;
  end
   else
        case(load)
            2'b00: prl_out <= prl_out;      // hold
            2'b01: prl_out <= {prl_out[2:0], 1'b0};  // shift left
            2'b10: prl_out <= {1'b0, prl_out[3:1]};  // shift right
            2'b11: prl_out <= prl_in;       // parallel load
            default: prl_out <= prl_out;    // safety: hold
        endcase
end  
endmodule
```
## [Test bench]
```bash
module Universal_shift_tb;
// Inputs
	reg clk;
	reg reset;
	reg [1:0]load;
	reg [3:0] prl_in;
	// Outputs
	//reg shift_left;
	//reg shift_right;
	wire [3:0] prl_out;
	// Instantiate the Unit Under Test (UUT)
	Universal_shift uut (
		.clk(clk), 
		.reset(reset), 
		.load(load),  
		.prl_in(prl_in), 
		.prl_out(prl_out)
	);  //.shift_left(shift_left), 
		//.shift_right(shift_right), 
	  always #5 clk=~clk;
	initial begin
		// Initialize Inputs
		clk = 0;
		reset = 1;
		load = 2'b00;
		//shift_left=0;
		//shift_right=0;
		prl_in = 4'b0000;
		// Wait 100 ns for global reset to finis
		#5 reset = 1;
      #10 reset = 0;
    // Load parallel data
    prl_in = 4'b1010;
    load = 2'b01;
    #10; 
	 load =2'b10; 
	 #10;
    load = 2'b11;
    #10; 
	 load =2'b00;
    #10;
    load = 2'b01;
    #10; 
	 load =2'b10; 
	 #10;
    load = 2'b11;
    #10; 
	 load =2'b00;	 // After one clock, data loaded
	end
     initial begin
      #200 $finish;
		end
		initial begin
   $monitor("clk=%b,reset=%b,load=%b prl_in=%b,prl_out=%b",clk,reset,load,prl_in,prl_out);
	//Step3 : Use $monitor to display the various inputs and outputs
      end 
endmodule
```
# sync up(4-bit)
## [RTL]
```bash
module four_counter(clk,reset,load,data,count);
input clk,reset,load;
input [3:0]data;
output reg[3:0]count;
always @(posedge clk)
begin
if(reset)
begin
count<=0;
end
else if(load)
begin
count<=data;
end
else
begin
count<=count+1;
end
end
endmodule
```
## [Test bench]
```bash
module four_counter_tb;
	reg clk;
	reg reset;
	reg load;
	reg [3:0] data;
	wire [3:0] count;
   integer i;
	// Instantiate the Unit Under Test (UUT)
	four_counter uut (.clk(clk), .reset(reset), .load(load), .data(data), .count(count));
	always
	  begin
	 clk=1'b0;
	 #10;
	 clk=1'b1;
	 #10;
	 end
	 task re_set;
	 begin
	 @(negedge clk);
	  reset=1'b1;
	 @(negedge clk);
	  reset=1'b0;
	  end
    endtask	 
    task in(input [3:0]k);
     begin
       data=k;
      end
    endtask	
	 task load_(input a);
	 begin
	   load=a;
	 end
	 endtask
	initial begin
		// Initialize Inputs
		clk = 0;
		reset = 0;
		load = 0;
		data = 0;
		#10;
		re_set;
       load_(0);
		in(4'b0001);
		re_set;
		load_(1);
		in(4'b1001);
		re_set;
		load_(1);
		in(4'b0001);
		re_set;
	   load_(0);
		in(4'b1001);
		re_set;
      load_(1);
		in(4'b1101);
		#10;
		re_set;
        load_(1);
		end
	end
	initial begin
      #1000$finish;
		end
		initial begin
   $monitor("clk=%b,reset=%b,data=%b,load=%b,count=%b",clk,reset,data,load,count);
	end  
endmodule
```
# mod12_upcounter
## [RTL]
```bash
module mod12_upcounter(reset,clk,load,data,count,);
input reset,clk,load;
input [3:0]data;
output reg[3:0]count;
always @(posedge clk)
begin 
if(reset)
  begin
   count<=4'b0000;
	end
else if(count==4'b1100)
   begin
   count<=4'b0000;
	end
else if(load)
   begin 
	 count<=data;
	end
else
  begin
   count<=count+1;
	end
end 
endmodule
```
## [Test bench]
```bash
module mod12_upcounter_tb;
	reg reset;
	reg clk;
	reg load;
	reg [3:0] data;
	wire [3:0] count;
   integer i;
	// Instantiate the Unit Under Test (UUT)
	mod12_upcounter uut (.reset(reset), .clk(clk), .load(load), .data(data), .count(count));
      always
	  begin
	 clk=1'b0;
	 #10;
	 clk=1'b1;
	 #10;
	 end
	 task re_set;
	 begin
	 @(negedge clk);
	  reset=1'b1;
	 @(negedge clk);
	  reset=1'b0;
	  end
    endtask	 
    task in(input [3:0]k);
     begin
       data=k;
      end
    endtask	
	 task load_(input a);
	 begin
	   load=a;
	 end
	 endtask
	initial begin
		clk = 0;
		reset = 0;
		load = 0;
		data = 0;
		#10;
		re_set;
      load_(0);
		in(4'b0001);
		re_set;
		load_(1);
		in(4'b1001);
		re_set;
		load_(1);
		in(4'b0001);
		re_set;
	   load_(0);
		in(4'b1001);
		re_set;
      load_(1);
		in(4'b1100);
		#10;
		end
	end
       initial begin
      #500 $finish;
		end
		initial begin
   $monitor("clk=%b,reset=%b,load=%b,data=%b,count=%b",clk,reset,load,data,count);
      end
endmodule
```
# bidirectional up-down
## [RTL]
```bash
module up_downcounter(clk,reset,load,data,dir,count);
input clk,reset,dir,load;
input [3:0]data;
output reg[3:0]count;
always @(posedge clk)
begin
if(reset)
begin
count<=0;
end
else if(load)
begin
count<=data;
end
else if(dir)
begin
count<=count+1;
end
else
begin
count<=count-1;
end
end
endmodule
```
## [Test bench]
```bash
module up_downcounter_tb;
	reg clk;
	reg reset;
	reg load;
	reg [3:0] data;
	reg dir;
	wire [3:0] count;
	up_downcounter uut (.clk(clk), .reset(reset), .load(load), .data(data), .dir(dir), .count(count));
	always
	  begin
	 clk=1'b0;
	 #10;
	 clk=1'b1;
	 #10;
	 end
      task re_set;
	 begin
	 @(negedge clk);
	  reset=1'b1;
	 @(negedge clk);
	  reset=1'b0;
	  end
    endtask	 
    task in(input [3:0]k);
     begin
       data=k;
      end
    endtask	
	 task load_(input a);
	 begin
	   load=a;
	 end
	 endtask
	 task dirr(input w);
	 begin
	  dir=w;
	 end
	 endtask
	initial begin
		clk = 0;
		reset = 0;
		load = 0;
		data = 0;
		dir = 0;
		#100;
		dirr(1);
		re_set;
		load_(1);
		in(4'b0001);
		dirr(0);
		re_set;
	   load_(0);
		in(4'b1001);
		in(4'b1101);
		#10;
	end
      initial begin
      #600 $finish;
		end
		initial begin
   $monitor("clk=%b,reset=%b,data=%b,load=%b,dir=%b;count=%b",clk,reset,data,load,dir,count);
      end  
endmodule
```
# 3_bit asyn up counter posedge
## [RTL]
```bash
//3 bit asynchronous up counter using posedge triggerd T-FF
module threebit_asyn_up_counter(t0,t1,t2,reset,clk,Q0,Q1,Q2,qb0,qb1,qb2);
input t0,t1,t2,clk,reset;
wire q0,q1,q2;
output qb0,qb1,qb2;
output Q0,Q1,Q2;

tff a1(.clk(clk),.reset(reset),.t(t0),.q(q0),.qb(qb0));
tff a2(.clk(qb0),.reset(reset),.t(t1),.q(q1),.qb(qb1));
tff a3(.clk(qb1),.reset(reset),.t(t2),.q(q2),.qb(qb2));
assign Q0=q0;
assign Q1=q1;
assign Q2=q2;

endmodule
module tff(clk,reset,t,q,qb);
input clk,t,reset;
output reg q;
output qb;
always@(posedge clk or posedge reset)
if(reset)
begin
q<=1;
end
else
begin
 case(t)
  0:q<=q;
  1:q<=~q;
  endcase
end
assign qb=~q;
endmodule
```
## [Test bench]
```bash
module threebit_asyn_up_counter_tbb;

	// Inputs
	reg t0;
	reg t1;
	reg t2;
	reg reset;
	reg clk;

	// Outputs
	wire Q0;
	wire Q1;
	wire Q2;
	wire qb0;
	wire qb1;
	wire qb2;
   integer i;
	// Instantiate the Unit Under Test (UUT)
	threebit_asyn_up_counter uut (
		.t0(t0), 
		.t1(t1), 
		.t2(t2), 
		.reset(reset), 
		.clk(clk), 
		.Q0(Q0), 
		.Q1(Q1), 
		.Q2(Q2), 
		.qb0(qb0), 
		.qb1(qb1), 
		.qb2(qb2)
	);
	   always
	  begin
	 clk=1'b0;
	 #10;
	 clk=1'b1;
	 #10;
	 end
		 task re_set;
	 begin
	 @(negedge clk);
	  reset=1'b1;
	 
	 @(negedge clk);
	  reset=1'b0;
	  end
    endtask	
	initial begin
		// Initialize Inputs
		t0 = 0;
		t1 = 0;
		t2 = 0;
		reset = 0;
		clk = 0;
		#10;
        re_set;
		  t0=1;
		  t1=1;
		  t2=1;
		  #20;
	end
	initial begin
	#500 $finish;
	end
	initial begin 
	$monitor(" t0=%b,t1=%b,t2=%b, reset=%b clk=%b Q0=%b,Q1=%b,Q2=%b,qb0=%b,qb1=%b,qb2=%b",t0,t1,t2,reset,clk,Q0,Q1,Q2,qb0,qb1,qb2);
    end  
endmodule
```
# 3_bit asyn down counter posedge
## [RTL]
```bash
//3 bit asynchronous down counter using posedge triggerd T-FF
module threebit_asyn_down_counter(t0,t1,t2,reset,clk,Q0,Q1,Q2,qb0,qb1,qb2);
input t0,t1,t2,clk,reset;
wire q0,q1,q2;
output qb0,qb1,qb2;
output Q0,Q1,Q2;

tff a1(.clk(clk),.reset(reset),.t(t0),.q(q0),.qb(qb0));
tff a2(.clk(q0),.reset(reset),.t(t1),.q(q1),.qb(qb1));
tff a3(.clk(q1),.reset(reset),.t(t2),.q(q2),.qb(qb2));
assign Q0=q0;
assign Q1=q1;
assign Q2=q2;

endmodule
module tff(clk,reset,t,q,qb);
input clk,t,reset;
output reg q;
output qb;
always@(posedge clk)
if(reset)
begin
q<=1;
end
else
begin
 case(t)
  0:q<=q;
  1:q<=~q;
  endcase
end
assign qb=~q;
endmodule
```
## [Test bench]
```bash
module threebit_asyn_down_counter_tbb;
	// Inputs
	reg t0;
	reg t1;
	reg t2;
	reg reset;
	reg clk;
	// Outputs
	wire Q0;
	wire Q1;
	wire Q2;
	wire qb0;
	wire qb1;
	wire qb2;
   integer i;
	// Instantiate the Unit Under Test (UUT)
	threebit_asyn_down_counter uut (
		.t0(t0), 
		.t1(t1), 
		.t2(t2), 
		.reset(reset), 
		.clk(clk), 
		.Q0(Q0), 
		.Q1(Q1), 
		.Q2(Q2), 
		.qb0(qb0), 
		.qb1(qb1), 
		.qb2(qb2)
	);
	   always
	  begin
	 clk=1'b0;
	 #10;
	 clk=1'b1;
	 #10;
	 end
		 task re_set;
	 begin
	 @(negedge clk);
	  reset=1'b1;
	 
	 @(negedge clk);
	  reset=1'b0;
	  end
    endtask	
	initial begin
		// Initialize Inputs
		t0 = 0;
		t1 = 0;
		t2 = 0;
		reset = 0;
		clk = 0;
		#10;
        re_set;
		  t0=1;
		  t1=1;
		  t2=1;
		  #20;
	end
	initial begin
	#500 $finish;
	end
	initial begin 
	$monitor(" t0=%b,t1=%b,t2=%b, reset=%b clk=%b Q0=%b,Q1=%b,Q2=%b,qb0=%b,qb1=%b,qb2=%b",t0,t1,t2,reset,clk,Q0,Q1,Q2,qb0,qb1,qb2);
    end  
endmodule
```
# Ring counter
## Method 1
### [RTl]
```bash
module ring_counter(clk,pr0,pr1,pr2,pr3,clr0,clr1,clr2,clr3,q0,q1,q2,q3,qb0,qb1,qb2,qb3);
input clk;
input  pr0,pr1,pr2,pr3,clr0,clr1,clr2,clr3;
output q0,q1,q2,q3,qb0,qb1,qb2,qb3;
wire a,b,c,e;
dff d1(.clk(clk),.clr(clr0),.pre(pr0),.d_in(e),.Q_out(a),.Qb_out(qb0));
dff d2(.clk(clk),.clr(clr1),.pre(pr1),.d_in(a),.Q_out(b),.Qb_out(qb1));
dff d3(.clk(clk),.clr(clr2),.pre(pr2),.d_in(b),.Q_out(c),.Qb_out(qb2));
dff d4(.clk(clk),.clr(clr3),.pre(pr3),.d_in(c),.Q_out(e),.Qb_out(qb3));
assign q0=a;
assign q1=b;
assign q2=c;
assign q3=e;
endmodule
module dff(clk,clr,pre,d_in,Q_out,Qb_out);
   input clk,d_in,clr,pre;
	output reg Q_out;
	output Qb_out;
   always@(posedge clk or posedge clr or posedge pre)
      begin
	 if(clr)
	    Q_out <= 1'b0;
	 else if(pre)
	    Q_out <= 1'b1;
	 else
	    Q_out <= d_in;
      end			
	 assign Qb_out=~(Q_out); 
endmodule  
```
## [Test bench]
```bash
module ring_counter_tb;
	reg clk;
	reg pr0;
	reg pr1;
	reg pr2;
	reg pr3;
	reg clr0;
	reg clr1;
	reg clr2;
	reg clr3;
	wire q0;
	wire q1;
	wire q2;
	wire q3;
	wire qb0;
	wire qb1;
	wire qb2;
	wire qb3;
	ring_counter uut (
		.clk(clk), 
		.pr0(pr0), 
		.pr1(pr1), 
		.pr2(pr2), 
		.pr3(pr3), 
		.clr0(clr0), 
		.clr1(clr1), 
		.clr2(clr2), 
		.clr3(clr3), 
		.q0(q0), 
		.q1(q1), 
		.q2(q2), 
		.q3(q3), 
		.qb0(qb0), 
		.qb1(qb1), 
		.qb2(qb2), 
		.qb3(qb3)
	);
	 always 
	 begin
	 clk=1'b0;
	 #10;
	 clk=1'b1;
	 #10;
	 end
	initial begin
       clk = 0;
    pr0 = 0; pr1 = 0; pr2 = 0; pr3 = 0;
    clr0 = 1; clr1 = 1; clr2 = 1; clr3 = 1; // clear all first
    #20;
    clr0 = 0; clr1 = 0; clr2 = 0; clr3 = 0;
    pr3 = 1;
    #10;pr3 = 0;
	end
       initial begin
      #500 $finish;
		end
		initial begin
$monitor("clk=%b,pr0=%b,pr1=%b,pr2=%b,pr3=%b,clr0=%b,clr1=%b,clr2=%b,clr3=%b,q0=%b,q1=%b,q2=%b,q3=%b,qb0=%b,qb1=%b,qb2=%b,qb3=%b",clk,pr0,pr1,pr2,pr3,clr0,clr1,clr2,clr3,q0,q1,q2,q3,qb0,qb1,qb2,qb3);
      end
endmodule
```
## Method 2
### [RTL]
```bash
module ring_counter(clk,count,reset);  
parameter N=6;
input clk,reset;
output reg [N-1:0]count;
always @(posedge clk)
if(reset)
begin
 count=1;
end
else 
begin
count={count[0],count[N-1:1]}; 
end
endmodule
```
## [Test bench]
```bash
module ring_counter_tb;
parameter N=6;
reg clk;
reg reset;
wire [N-1:0]count;
ring_counter dut(.clk(clk),.reset(reset),.count(count));
always #5 clk=~clk;
initial 
begin
clk=0;
reset=1;
#10;
reset=0;
end
initial begin
      #500 $finish;
		end
		initial begin
		$monitor("clk=%b reset=%b count=%b",clk,reset,count);
		end
endmodule
```
# Johnson counter
## Method 1
### [RTL]
```bash
module johnson_counter(clk,pr0,pr1,pr2,pr3,clr0,clr1,clr2,clr3,q0,q1,q2,q3,qb0,qb1,qb2,qb3);
input clk;
input  pr0,pr1,pr2,pr3,clr0,clr1,clr2,clr3;
output q0,q1,q2,q3,qb0,qb1,qb2,qb3;
wire a,b,c,e;
dff d1(.clk(clk),.clr(clr0),.pre(pr0),.d_in(~e),.Q_out(a),.Qb_out(qb0));
dff d2(.clk(clk),.clr(clr1),.pre(pr1),.d_in(a),.Q_out(b),.Qb_out(qb1));
dff d3(.clk(clk),.clr(clr2),.pre(pr2),.d_in(b),.Q_out(c),.Qb_out(qb2));
dff d4(.clk(clk),.clr(clr3),.pre(pr3),.d_in(c),.Q_out(e),.Qb_out(qb3));
assign q0=a;
assign q1=b;
assign q2=c;
assign q3=e;
endmodule
module dff(clk,clr,pre,d_in,Q_out,Qb_out);
   input clk,d_in,clr,pre;
	output reg Q_out;
	output Qb_out;
   always@(posedge clk or posedge clr or posedge pre)
      begin
	 if(clr)
	    Q_out <= 1'b0;
	 else if(pre)
	    Q_out <= 1'b1;
	 else
	    Q_out <= d_in;
      end			
	 assign Qb_out=~(Q_out); 
endmodule 
```
### [Test bench]
```bash
module johnson_counter_tb;
	reg clk;
	reg pr0;
	reg pr1;
	reg pr2;
	reg pr3;
	reg clr0;
	reg clr1;
	reg clr2;
	reg clr3;
	wire q0;
	wire q1;
	wire q2;
	wire q3;
	wire qb0;
	wire qb1;
	wire qb2;
	wire qb3;
	johnson_counter uut (
		.clk(clk), 
		.pr0(pr0), 
		.pr1(pr1), 
		.pr2(pr2), 
		.pr3(pr3), 
		.clr0(clr0), 
		.clr1(clr1), 
		.clr2(clr2), 
		.clr3(clr3), 
		.q0(q0), 
		.q1(q1), 
		.q2(q2), 
		.q3(q3), 
		.qb0(qb0), 
		.qb1(qb1), 
		.qb2(qb2), 
		.qb3(qb3)
	);
always 
	 begin
	 clk=1'b0;#10;
     clk=1'b1; #10;
	 end
	initial begin
   clk = 0;
    pr0 = 0; pr1 = 0; pr2 = 0; pr3 = 0;
    clr0 = 1; clr1 = 1; clr2 = 1; clr3 = 1; // clear all first
    #20;
    clr0 = 0; clr1 = 0; clr2 = 0; clr3 = 0;
    pr3 = 1;
    #10;pr3 = 0;
	end
       initial begin
      #500 $finish;
		end
		initial begin
$monitor("clk=%b,pr0=%b,pr1=%b,pr2=%b,pr3=%b,clr0=%b,clr1=%b,clr2=%b,clr3=%b,q0=%b,q1=%b,q2=%b,q3=%b,qb0=%b,qb1=%b,qb2=%b,qb3=%b",clk,pr0,pr1,pr2,pr3,clr0,clr1,clr2,clr3,q0,q1,q2,q3,qb0,qb1,qb2,qb3);
      end
endmodule
```
## Method 2
### [RTL]
```bash
module johnson_counter(clk,reset,q);
parameter size=4;
input clk,reset;
output reg [size-1:0]q;
always@(posedge clk)
begin
  if(reset)
    q<=1;
	else 
	   q<={~q[0],q[size-1:1]};
end 
endmodule 
```
### [Test bench]
```bash
module johnson_counter_tb;
parameter size=4;
reg clk;
reg reset;
wire [size-1:0]q;
johnson_counter uut(.clk(clk),.reset(reset),.q(q));
always #5 clk=~clk;
 initial begin
 clk=0;
 reset=1;
  #10;
 reset=0;
 end
 initial begin
      #500 $finish;
		end
		initial begin
   $monitor("clk=%b,reset=%b,q=%b",clk,reset,q);
      end
endmodule 
```
## Gray code counter
### [RTL]
```bash
//`define 3bit
`define n_bit
`ifdef 3bit
module graycode_counter(clk,reset,in,out);
input clk,reset;
input [2:0]in;
output reg [2:0]out;
`endif
`ifdef n_bit
module graycode_counter(clk,reset,out);
parameter N=5;
input clk,reset;
output reg [N-1:0]out;
 `endif
`ifdef 3bit
always@(posedge clk)
begin
  if(reset)
    out<=3'b000;
  else
    out<={in[2],in[2]^in[1],in[1]^in[0]};
end

`elsif n_bit 
reg [N-1:0]in;
always@(posedge clk or posedge reset)
begin
  if(reset)
    in<={N{1'b0}};
	 else
	in<=in+1;
end
always@(*)
begin
    out<=in^(in>>1);
end
`endif
endmodule
```
### [Test bench]
```bash
//`define 3bit
`define n_bit
module graycode_counter_tb;
`ifdef 3bit
	// Inputs
	reg clk;
	reg reset;
	reg [2:0]in;
	// Outputs
	wire [2:0]out;
   integer i;
	graycode_counter uut (
		.clk(clk), 
		.reset(reset),
      .in(in),		
		.out(out)
	`endif
	`ifdef n_bit
	// Inputs
	parameter N=4;
	reg clk;
	reg reset;
	wire [N-1:0]out;
	// Instantiate the Unit Under Test (UUT)
	graycode_counter uut (
		.clk(clk), 
		.reset(reset), 
		.out(out)
	);
`endif
   always #5 clk=~clk;
	`ifdef 3bit
	initial begin
		// Initialize Inputs
		clk = 0;
		reset = 1;
		in = 0;
		// Wait 100 ns for global reset to finish
		#100;
        reset=0;
		 for(i=0;i<8;i=i+1)
		 begin
		 in=i;
		  #10;
		  end
	end
	initial begin
      #500 $finish;
		end
		initial begin
   $monitor("clk=%b,reset=%b,in=%b out=%b",clk,reset,in,out);
      end 
	`elsif n_bit
	initial begin
		// Initialize Inputs
		clk = 0;
		reset = 1;
		#10;
        reset=0;
		  #10;
	end
	initial begin
      #500 $finish;
		end
		initial begin
   $monitor("clk=%b,reset=%b,out=%b",clk,reset,out);
      end 
		`endif
endmodule

```








