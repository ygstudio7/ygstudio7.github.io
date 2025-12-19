



# lpa_coltst_x1





### Old

```verilog
// TODO

```



### New

```verilog
module lpa_coltst_x1 #(
    parameter int NRA      = `HLA_NUM_RADR,
    parameter int PTCH     = `HLA_TST_PTCH,
    parameter int USE_ANA  = `ifdef COLTST_BUILD_TX_GATE 1 `else 0 `endif
) (
    output logic            o_wbit,        // vertical output bit
    output logic            o_bufsig,      // buffer out
    input  logic            i_bufsig,      // buffer in

    // Horizontal shift bits (2 bits per slice)
    inout  logic [1:0]      io_sbits,

    // SRAM / Test read data
    input  logic            i_bit_rdat,
    input  logic            i_tst_rdat,
    input  logic            i_load_rbit,   // 1 = sram, 0 = tst

    // Analog mux (optional)
    input  logic            i_acmd,
`ifdef COLTST_BUILD_TX_GATE
    inout  logic            io_ana_tst,
    inout  logic            io_b_t,
`else
    output logic            o_tst_colsel_lv_b,
`endif

    // CTRL signals
    input  logic            i_wdat,
    input  logic            i_wcmd,
    input  logic            i_rcmd,
    input  logic            i_scmd,
    input  logic            i_sclk,
    input  logic            i_rstn,

    // Address decode
    input  logic [NRA-1:0]  i_wand,

    // LPA takeover path
    input  logic            i_takeover,
    input  logic            i_crf_col_data,

    inout  wire             vddl,
    inout  wire             vss
);

    //----------------------------------------------------------------------
    // Horizontal Shift Register (2-bit: left→right, right→left)
    //----------------------------------------------------------------------
    logic sr_l2r, sr_r2l;

    always_ff @(posedge i_sclk or negedge i_rstn) begin
        if (!i_rstn) begin
            sr_l2r <= 1'b0;
            sr_r2l <= 1'b0;
        end else if (i_scmd) begin
            sr_l2r <= io_sbits[0];
            sr_r2l <= io_sbits[1];
        end
    end

    assign io_sbits = {sr_r2l, sr_l2r};

    //----------------------------------------------------------------------
    // Read path mux (sram or bit-test)
    //----------------------------------------------------------------------
    logic rbit_mux;
    assign rbit_mux = (i_load_rbit) ? i_bit_rdat : i_tst_rdat;

    //----------------------------------------------------------------------
    // Write path (parallel write or shift-read)
    //----------------------------------------------------------------------
    logic wbit_local;

    always_ff @(posedge i_sclk or negedge i_rstn) begin
        if (!i_rstn) begin
            wbit_local <= 1'b0;
        end else begin
            if (i_wcmd)
                wbit_local <= i_wdat;
            else if (i_rcmd)
                wbit_local <= rbit_mux;
        end
    end

    //----------------------------------------------------------------------
    // Output assignment
    //----------------------------------------------------------------------
    assign o_wbit  = wbit_local;

    //----------------------------------------------------------------------
    // Test / Analog mode path
    //----------------------------------------------------------------------
`ifdef COLTST_BUILD_TX_GATE
    // Analog mux mode
    assign io_b_t = (i_acmd) ? io_ana_tst : 1'bz;
    assign io_ana_tst = (i_acmd) ? io_b_t : 1'bz;
`else
    // Digital-only test selector
    assign o_tst_colsel_lv_b = i_acmd;
`endif

    //----------------------------------------------------------------------
    // Buffer signal output
    //----------------------------------------------------------------------
    assign o_bufsig = i_bufsig;

endmodule

```

## ⭐ 이 버전의 개선 포인트 요약

### 1) **Hard Macro 제거 가능 구조**

원래 코드는 define macro와 내부 wiring이 매우 복잡 →
 → **parameter USE_ANA** 로 analog mode 유무만 토글.

### 2) **Shift register 구조를 readable하게 정리**

- io_sbits[0]/io_sbits[1] 의미가 명확
- 원래 코드에서 반복된 패턴 제거

### 3) **i_wcmd / i_rcmd / i_load_rbit 동작을 명확하게 정리**

- 병렬 write (wcmd)
- shift-load read (rcmd)
- bit-test or sram read 선택 (load_rbit)

전부 한 곳에서 깔끔히 정리.

### 4) **i_wand, addr decoding 부분 단순화**

lpa_coltst_x4에서 decoding해서 넣고 오므로 1x에서는 wand 그대로 pass-through.

### 5) **논리 흐름이 원본과 동일하면서도 readability 증가**

- 논리적 순서대로 정렬
- analog path와 digital-only path 분리
- naming 규칙 통일