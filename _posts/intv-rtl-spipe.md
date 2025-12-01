```verilog
//******************************************************************************
//	Copyright 2024 ImagingReal, Inc. All right reserved. 
// The reproduction, distribution, display, or transmission of the content is 
// strictly prohibited, unless authorized by the ImagingReal, Inc.
//
//	File Name:		im_pipe.sv
//	Description:	Sync pipe
//	Author:			ykim@imagingreal.com
//	Note:
// -----------------------------------------------------------------------------
module im_spipe #(
	parameter 					AW				= 1,
	parameter type				data_t		= logic,
	parameter type				addr_t		= logic [AW-1:0]
) (                                 	
	input logic   				CLK  			,
	input logic   				RSTN 			,
	input logic					FLUSH			,
	                                 	
	im_pipe_intf.sp			IN				,
	im_pipe_intf.mp 			OUT			,
	                                 	
	output logic [AW:0]		COUNT
);
	localparam					DEPTH 		= 2**AW;

	logic 						wfull			; 
	logic 						rempty		;	
	logic [AW:0]				count			;

	im_sfifo #(
		.AW	        			(AW			),
		.data_t       			(data_t		)	
	) U_FIFO (
		.CLK						(CLK			),
		.RSTN						(RSTN			),
		.FLUSH   				(FLUSH		),	
		            			
		.WDATA    				(IN.DATA    ),
		.WRITE    				(IN.VALID   ),
		.WFULL    				(wfull	   ),
		            				
		.RDATA    				(OUT.DATA  	),
		.REMPTY   				(rempty    	),
		.READ     				(OUT.READY  ),
		            			
		.COUNT  					(count		)
	);

//******************************************************************************
// Outputs
//------------------------------------------------------------------------------
//
	assign						IN.READY 	= ~wfull;
	assign 						OUT.VALID 	= ~rempty;
	assign						COUNT			= count;

endmodule
```

