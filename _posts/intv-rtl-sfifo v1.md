```verilog
//******************************************************************************
//	Copyright 2024 ImagingReal, Inc. All right reserved. 
// The reproduction, distribution, display, or transmission of the content is 
// strictly prohibited, unless authorized by the ImagingReal, Inc.
//
//	File Name:		im_sfifo.sv
//	Description:	Sync FIFO
//	Author:			ykim@imagingreal.com
//	Note:
// -----------------------------------------------------------------------------
module im_sfifo #(
	parameter 					AW			= 4,
	parameter type				data_t	= logic,
	//	          					
	parameter type				addr_t	= logic [AW-1:0]
) (
    input logic   			CLK  		,
    input logic   			RSTN 		,
    input logic				FLUSH		,
    
    input data_t  			WDATA  	,
    input logic   			WRITE		,
    output logic    			WFULL 	,
    
    output data_t     		RDATA 	,
    output logic    			REMPTY	,
    input logic   			READ 		,
    
    output logic [AW:0]		COUNT
);
	localparam 					DEPTH  	= 2**AW;

	addr_t 						Wptr, Rptr;
	data_t [DEPTH-1:0]		MEM;
	logic [AW:0] 				Count;
	
	logic              		full;		assign full 	= (Count==(AW+1)'(DEPTH));
	logic              		empty;	assign empty 	= (Count==(AW+1)'('0));
	logic              		wfire; 	assign wfire 	= WRITE & ~full;
	logic              		rfire; 	assign rfire 	=  READ & ~empty;
	logic							wend;		assign wend 	= (Wptr == addr_t'(DEPTH-1));
	logic							rend;		assign rend 	= (Rptr == addr_t'(DEPTH-1));
	
	always_ff @ (posedge CLK or negedge RSTN) begin
		if(!RSTN) begin         
			Wptr 	<= '0;
			Rptr 	<= '0;      
			Count	<= '0;
		end
		else if(FLUSH) begin
			Wptr 	<= '0;
			Rptr 	<= '0;      
			Count	<= '0;
		end			
		else begin			
			if(rfire) begin         			
				if(rend) 	Rptr <= '0;      
				else      	Rptr <= Rptr + 1;
			end
      
			if(wfire) begin
				if(wend) 	Wptr <= '0;
				else			Wptr <= Wptr + 1;
      		MEM[Wptr] <= WDATA;
			end
			
			if(~rfire&wfire) 			Count <= Count + 1;
         else if(rfire&~wfire) 	Count <= Count - 1;         
		end
	end


//******************************************************************************
// Outputs
//------------------------------------------------------------------------------
//
	assign						WFULL = full;
	assign						RDATA = MEM[Rptr];
   assign						REMPTY = empty;
   assign						COUNT  = Count;

endmodule
```

