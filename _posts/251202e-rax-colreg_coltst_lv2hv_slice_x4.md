# colreg_coltst_lv2hv_slice_x4



### Old

```verilog


module	colreg_coltst_lv2hv_slice_x4
			#(parameter
			nra=`HLA_NUM_RADR,
			ctw=`HLA_TST_CTW,
			ctr=`HLA_TST_CTR,
			ptch=`HLA_TST_PTCH,
			npa=$clog2(ptch)
			)
			(

  		output	[0:ptch-1]	o_tst_colsel_hv_b,

		output	[0:ptch-1]	o_wbit,		// verital output to coldrvp
		output	[0:ptch-1]	o_bufsig,
		input	[0:ptch-1]	i_bufsig,
`ifdef	SIMULATION
// 		input			i_a2b,		// when high, then io_ana_tst is an input, drive io_b_t.  when low, reverse
`endif//SIMULATION
		inout	[0:(2*ptch)-1]	io_sbits,	// horizontal right2left
		input	[0:ptch-1]	i_bit_rdat,	// from bit sense amp
		input	[0:ptch-1]	i_tst_rdat,	// from bitline test sense amp
		input	[0:ptch-1]	i_load_rbit,	// 1== sram rdata, 0== csrc rdata

		input   [nra-1:npa]     i_wand,         // address for parallel writes or io_ana selection
		input   [npa-1:0]       i_badrs_p,
		input   [npa-1:0]       i_badrs_n,
		//input	[nra-1:3]	i_wand,		// address for parallel writes or io_ana selection
		input	[0:ptch-1]	i_wdat,		// data for parallel writes
		input	[0:ptch-1]	i_acmd,		// enable mux io_b_t onto/from io_ana_tst
		input	[0:ptch-1]	i_wcmd,		// parallel write command (when selected) from i_wdat from lpa_tester
		input	[0:ptch-1]	i_rcmd,		// parallel shift reg load from lpa (sram rbit or tst rbit)
		input	[0:ptch-1]	i_scmd,		// shift command (shifts on posedge clk if i_scmd is high)

		input	[0:ptch-1]	i_takeover,
		input	[0:ptch-1]	i_sclk,
		input	[0:ptch-1]	i_rstn,

		input			cclk,
		input			reset,
		input		[0:3]	i_data, 
		input			read, upd,
		input			set, clr, 
		output	reg	[0:3]	r_data,

		inout		vddl,
		inout		vddar,
		inout		vss
	);

  wire	[0:ptch-1]	w_tst_colsel_lv_b;

	lv2hv_buffers_x4	lv2hv_buffers_x4 (
		.i_lv_sigs	(w_tst_colsel_lv_b),
		.o_hv_sigs	(o_tst_colsel_hv_b),
		.v0p8	(vddl),
		.vddar	(vddar),
		.vss	(vss)
		);

	colreg_coltst_slice_x4 colreg_coltst_slice_x4 (
		.o_wbit			(o_wbit),
		.o_tst_colsel_lv_b	(w_tst_colsel_lv_b),
		.o_bufsig		(o_bufsig),
		.i_bufsig		(i_bufsig),
	`ifdef	SIMULATION
//		.i_a2b			(i_a2b),
	`endif//SIMULATION
		.io_sbits		(io_sbits),
		.i_bit_rdat		(i_bit_rdat),
		.i_tst_rdat		(i_tst_rdat),
		.i_load_rbit		(i_load_rbit),
		.i_wand			(i_wand),
		.i_badrs_p		(i_badrs_p),
		.i_badrs_n		(i_badrs_n),
		.i_wdat			(i_wdat),
		.i_acmd			(i_acmd),
		.i_wcmd			(i_wcmd),
		.i_rcmd			(i_rcmd),
		.i_scmd			(i_scmd),

		.i_takeover		(i_takeover),
		.i_sclk			(i_sclk),
		.i_rstn			(i_rstn),

		.cclk			(cclk),
		.reset			(reset),
		.i_data			(i_data),
		.upd			(upd),
		.clr			(clr),
		.read			(read),
		.set			(set),
		.r_data			(r_data),

		.vddl			(vddl),
		.vss			(vss)
	);

endmodule



```





### New

```verilog
```



