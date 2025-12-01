````verilog
//******************************************************************************
//  Copyright(c) Young-Geun Kim 2003-.
//
//  File Name:      sfifo_logic.v
//  Description:    sfifo_logic simulation model
//  Author:         Young-Geun Kim (ygkim@{[rcv.kaist.ac.kr](http://rcv.kaist.ac.kr/);[eyenix.com](http://eyenix.com/)})

//
//  Note:           Simulation only! Use the same Altera scfifo ("alt_asfifo.v")
//                  : WREQ to REMPTY : latency =1, but 2 @ alt_scfifoAxD.v
//                  SHOW_AHEAD: =0 (RREQ: read acknowledge), =1 (RREQ: read request)
//                  Stop writing when WFULL=1
//                  Stop reading when REMPTY=1
//                  Use AWFULL and AREMPTY when an early warning is required (Set ALMOST_FULL_VALUE, ALMOST_EMPTY_VALUE)
//
//  Ver Date        Author      Changes
//  --- ----        ------      -------
//  0.1 110302      ygkim       first designed
//  0.2 110309      ygkim       simulation with modelsim
//  0.3 110329      ygkim       same as alt_sfifo
//	0.4 120105		ygkim		simplified
// -----------------------------------------------------------------------------
`timescale  1 ns/ 10 ps

module sfifo_logic (

RSTN       ,
CLK        ,
WREQ       ,
WDATA      ,
WFULL      ,
AWFULL     ,
RREQ       ,
RDATA      ,
REMPTY     ,
AREMPTY    ,
USEDW

);

// Constant parameters
parameter           ABITS               = 2;
parameter           DBITS               = 32;
parameter           SHOW_AHEAD          = 0;
parameter           ALMOST_FULL_VALUE   = 3;
parameter           ALMOST_EMPTY_VALUE  = 2;

```
parameter           EXP_FNAME           = "sfifo_exp.csv";

```

// Internal parameters
localparam          ASIZE              	= 2**ABITS;

// Port declaration
input               RSTN       ;
input               CLK        ;
input               WREQ       ;
input  [DBITS-1:0]  WDATA      ;
output              WFULL      ;
output              AWFULL     ;
input               RREQ       ;
output [DBITS-1:0]  RDATA      ;
output              REMPTY     ;
output              AREMPTY    ;
output [ABITS-1:0]	USEDW      ;

`ifdef RTL_FIFO
//******************************************************************************
// RTL Model
//------------------------------------------------------------------------------
// Internal variables
reg	[DBITS-1:0]		MEM[ASIZE-1:0]	/* synthesis ramstyle = "logic" */;

```
wire				wfull;
wire				rempty;
reg [ABITS:0]		Usedw;

```

//------------------------------------------------------------------------------
// init
integer i;
initial begin
for(i=0; i<ABITS; i=i+1) begin
//MEM[i] = 'dx;	
MEM[i] = 'd0;		// FIX @120319
end
end

//------------------------------------------------------------------------------
// Valid write and read signals
wire				vwreq = WREQ & ~wfull;
wire				vrreq = RREQ & ~rempty;

//------------------------------------------------------------------------------
// Write
reg	[ABITS-1:0]		Wptr;
always @(posedge CLK or negedge RSTN) begin
if(!RSTN)		Wptr <= 'd0;
else if(vwreq)	Wptr <= Wptr + 1'd1;
end

```
always @(posedge CLK) begin
	if(vwreq) 		MEM[Wptr] <= WDATA;
end

```

//------------------------------------------------------------------------------
// Read
reg [ABITS-1:0]		Rptr;
always @(posedge CLK or negedge RSTN) begin
if(!RSTN)		Rptr <= 'd0;
else if(vrreq)	Rptr <= Rptr + 1'd1;
end

```
wire [DBITS-1:0]	rdata = MEM[Rptr];

reg [DBITS-1:0]		Rdata;
always @(posedge CLK or negedge RSTN) begin
	if(!RSTN)		Rdata <= 'd0;
	else if(vrreq)	Rdata <= rdata;
end

```

//------------------------------------------------------------------------------
// Status
wire				vwreq_only =  vwreq&~vrreq;
wire				vrreq_only = ~vwreq& vrreq;

```
wire [ABITS:0]		usedw_next = 	vwreq_only ? 	Usedw + 1'd1	:
									vrreq_only ? 	Usedw - 1'd1	:
													Usedw			;
//reg [ABITS:0]		Usedw;
always @(posedge CLK or negedge RSTN) begin
	if(!RSTN)		Usedw <= 'd0;
	else			Usedw <= usedw_next;
end

assign				wfull = Usedw[ABITS];

reg					Awfull;
always @(posedge CLK or negedge RSTN) begin
	if(!RSTN)								Awfull <= 'd0;
	else if(usedw_next>=ALMOST_FULL_VALUE) 	Awfull <= 'd1;
	else									Awfull <= 'd0;
end

reg					Rempty;
always @(posedge CLK or negedge RSTN) begin
	if(!RSTN)								Rempty <= 'd1;
	else if(usedw_next=='d0)				Rempty <= 'd1;
	else									Rempty <= 'd0;
end

reg					Arempty;
always @(posedge CLK or negedge RSTN) begin
	if(!RSTN)								Arempty <= 'd1;
	else if(usedw_next<	ALMOST_EMPTY_VALUE)	Arempty <= 'd1;
	else									Arempty <= 'd0;
end

```

//------------------------------------------------------------------------------
// Output
generate
if(SHOW_AHEAD) 	assign	RDATA = rdata;
else			assign	RDATA = Rdata;
endgenerate
assign				rempty	= Rempty;

```
assign				WFULL 	= wfull;
assign 				USEDW 	= Usedw[ABITS-1:0];
assign				AWFULL 	= Awfull;
assign				REMPTY	= rempty;
assign				AREMPTY = Arempty;

```

endmodule
````

