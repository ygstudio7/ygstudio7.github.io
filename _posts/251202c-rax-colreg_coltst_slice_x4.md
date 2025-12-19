# col_slice



### Old

```verilog
module	colreg_coltst_slice_x4	#(parameter
			nra=`HLA_NUM_RADR,
			ctw=`HLA_TST_CTW,
			ctr=`HLA_TST_CTR,
			ptch=`HLA_TST_PTCH,
			npa=$clog2(ptch) ) (
	output	[0:ptch-1]	o_wbit,			// vertical wire output to coldrvp
	output	[0:ptch-1]	o_tst_colsel_lv_b,	// vertical wire output to lv2hv then to coldrvp
	output	[0:ptch-1]	o_bufsig,		// used in horizontal wire buffering in coltst
	input	[0:ptch-1]	i_bufsig,		// used in horizontal wire buffering in coltst
`ifdef	SIMULATION
// 	input			i_a2b,		// when high, then io_ana_tst is an input, drive io_b_t.  when low, reverse
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
	input		[0:3]	i_data, 	// from ppath
	input			read, upd,
	input			set, clr, 
	output	reg	[0:3]	r_data,

	inout		vddl,
	inout		vss
	);

  wire		[0:3]	w_crf_data;	// colreg data output routed into coltst for multiplexing to coldrv

      lpa_coltst_x4	lpa_coltst_x4	(
		.o_wbit		(o_wbit),		// verital output (colreg/coltst) up to coldrvp
`ifdef	COLTST_BUILD_TX_GATE
		.io_ana_tst	(io_ana_tst),		// input/output multiplexor for analog isrc/vf test
		.io_b_t		(io_b_t),		// bit line tests in analog mode
`else  // ! COLTST_BUILD_TX_GATE
		.o_tst_colsel_lv_b	(o_tst_colsel_lv_b),
`endif // ! COLTST_BUILD_TX_GATE
		.io_sbits	(io_sbits),	// horizontal right2left within coltst
		.i_bit_rdat	(i_bit_rdat),	// vertical wire - data from srambit sense amp in coldrv
		.i_tst_rdat	(i_tst_rdat),	// vertical wire - data from testbit sense amp in coldrv

		// horizontal inputs
`ifdef	SIMULATION
//	 	.i_a2b		(i_a2b),	
`endif//SIMULATION
		.i_load_rbit	(i_load_rbit),		// 1== sram rdata, 0== csrc rdata
		.i_wand 	(i_wand),
		.i_badrs_p	(i_badrs_p),
		.i_badrs_n	(i_badrs_n),
		.i_wdat		(i_wdat),
		.i_acmd		(i_acmd),
		.i_wcmd		(i_wcmd),
		.i_rcmd		(i_rcmd),
		.i_scmd		(i_scmd),
		.i_takeover	(i_takeover),
		.i_crf_col_data	(w_crf_data),
		.i_sclk		(i_sclk),
		.i_rstn		(i_rstn),
		.o_bufsig	(o_bufsig),
		.i_bufsig	(i_bufsig),

		.vddl		(vddl),
		.vss		(vss)
	);

	col_slicex4 col_slicex4 (
		.i_data		(i_data), 
		.read		(read), 
		.upd		(upd),
		.cclk		(cclk),
		.reset		(reset),
		.set		(set), 
		.clr		(clr), 
		.o_data		(w_crf_data),
		.r_data		(r_data)
	);

endmodule

```





### New

```verilog
module colreg_coltst_slice_x4 #(
    // address / coltst parameters
    parameter nra  = `HLA_NUM_RADR,
    // parameter ctw  = `HLA_TST_CTW, // currently unused in this module
    // parameter ctr  = `HLA_TST_CTR, // currently unused in this module
    parameter ptch = `HLA_TST_PTCH,
    parameter npa  = $clog2(ptch),

    // local col-slice width (colreg bit width per slice)
    parameter COL_SLICE_W = 4
) (
    // coltst side (per-ptch)
    output [0:ptch-1]        o_wbit,          // vertical wire output to coldrvp
    output [0:ptch-1]        o_tst_colsel_lv_b, // vertical wire output to lv2hv then to coldrvp
    output [0:ptch-1]        o_bufsig,        // used in horizontal wire buffering in coltst
    input  [0:ptch-1]        i_bufsig,        // used in horizontal wire buffering in coltst
`ifdef SIMULATION
    // input                  i_a2b,          // when high, then io_ana_tst is an input, drive io_b_t. when low, reverse
`endif // SIMULATION
    inout  [0:(2*ptch)-1]    io_sbits,        // horizontal right2left
    input  [0:ptch-1]        i_bit_rdat,      // from bit sense amp
    input  [0:ptch-1]        i_tst_rdat,      // from bitline test sense amp
    input  [0:ptch-1]        i_load_rbit,     // 1== sram rdata, 0== csrc rdata

    input  [nra-1:npa]       i_wand,          // address for parallel writes or io_ana selection
    input  [npa-1:0]         i_badrs_p,
    input  [npa-1:0]         i_badrs_n,
    //input [nra-1:3]        i_wand,          // legacy form

    input  [0:ptch-1]        i_wdat,          // data for parallel writes
    input  [0:ptch-1]        i_acmd,          // enable mux io_b_t onto/from io_ana_tst
    input  [0:ptch-1]        i_wcmd,          // parallel write command
    input  [0:ptch-1]        i_rcmd,          // parallel shift reg load from lpa
    input  [0:ptch-1]        i_scmd,          // shift command

    input  [0:ptch-1]        i_takeover,
    input  [0:ptch-1]        i_sclk,
    input  [0:ptch-1]        i_rstn,

    // colreg (ppath) side
    input                    cclk,
    input                    reset,
    input  [0:COL_SLICE_W-1] i_data,   // from ppath
    input                    read,
    input                    upd,
    input                    set,
    input                    clr,
    output reg [0:COL_SLICE_W-1] r_data,

    inout                    vddl,
    inout                    vss
);

    // colreg data output routed into coltst for multiplexing to coldrv
    wire [0:COL_SLICE_W-1] w_crf_data;

    lpa_coltst_x4 #(
        .nra (nra),
        .ptch(ptch),
        .npa (npa)
    ) u_lpa_coltst_x4 (
        .o_wbit        (o_wbit),          // vertical output (colreg/coltst) up to coldrvp
`ifdef COLTST_BUILD_TX_GATE
        .io_ana_tst    (io_ana_tst),      // input/output multiplexor for analog isrc/vf test
        .io_b_t        (io_b_t),          // bit line tests in analog mode
`else  // !COLTST_BUILD_TX_GATE
        .o_tst_colsel_lv_b (o_tst_colsel_lv_b),
`endif // !COLTST_BUILD_TX_GATE
        .io_sbits      (io_sbits),        // horizontal right2left within coltst
        .i_bit_rdat    (i_bit_rdat),      // from srambit sense amp in coldrv
        .i_tst_rdat    (i_tst_rdat),      // from testbit sense amp in coldrv

`ifdef SIMULATION
        // .i_a2b       (i_a2b),
`endif // SIMULATION
        .i_load_rbit   (i_load_rbit),     // 1== sram rdata, 0== csrc rdata
        .i_wand        (i_wand),
        .i_badrs_p     (i_badrs_p),
        .i_badrs_n     (i_badrs_n),
        .i_wdat        (i_wdat),
        .i_acmd        (i_acmd),
        .i_wcmd        (i_wcmd),
        .i_rcmd        (i_rcmd),
        .i_scmd        (i_scmd),
        .i_takeover    (i_takeover),
        .i_crf_col_data(w_crf_data),
        .i_sclk        (i_sclk),
        .i_rstn        (i_rstn),
        .o_bufsig      (o_bufsig),
        .i_bufsig      (i_bufsig),

        .vddl          (vddl),
        .vss           (vss)
    );

    col_slicex4 #(
        .COL_SLICE_W (COL_SLICE_W)
    ) u_col_slicex4 (
        .i_data   (i_data),
        .read     (read),
        .upd      (upd),
        .cclk     (cclk),
        .reset    (reset),
        .set      (set),
        .clr      (clr),
        .o_data   (w_crf_data),
        .r_data   (r_data)
    );

endmodule

```





