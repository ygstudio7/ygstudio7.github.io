# col_slice



### Old

```verilog
module col_slice (
	//input [0:HLA_COL_SLICE-1] i_data, 
	input cclk,
	input reset,
	input i_data, 
	input read, upd,
	input set, clr, 
	output o_data,
	output reg r_data
);

	reg wp_slice_buf;
	wire test;
	assign o_data = wp_slice_buf;

	assign test = wp_slice_buf;
	always @ (posedge cclk or negedge reset)begin
		if (!reset) begin
			r_data <= #SLD 1'b0;
			wp_slice_buf <= #SLD 1'b0;
		end 
		else begin
			if (clr) wp_slice_buf <= #SLD 1'b0;
			else if (set) wp_slice_buf <= #SLD 1'b1;

			if (upd) wp_slice_buf <= #SLD i_data; 
			else if (read) begin  //SL??? read might need to come from sense amp
				r_data <= #SLD wp_slice_buf; 
			end
		end
	end


endmodule 




```





### New

```verilog


```



