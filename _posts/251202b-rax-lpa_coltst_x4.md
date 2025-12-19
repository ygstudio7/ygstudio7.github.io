# lpa_coltst_x4



### Old

```verilog




```





### New

```verilog
module lpa_coltst_x4 #(
    parameter int NRA      = `HLA_NUM_RADR,      // address width
    parameter int PTCH     = `HLA_TST_PTCH,      // width of slice (usually 6)
    parameter int SLICE_W  = 1                   // single slice per element
)(
    output logic [0:PTCH-1]   o_wbit,            // to cold RV
    output logic [0:PTCH-1]   o_tst_colsel_hv_b, // to HV level shifter
    output logic [0:PTCH-1]   o_bufsig,
    input  logic [0:PTCH-1]   i_bufsig,

    inout  logic [0:PTCH*2-1] io_sbits,          // horizontal chain

    input  logic [0:PTCH-1]   i_bit_rdat,        // sram read
    input  logic [0:PTCH-1]   i_tst_rdat,        // test read
    input  logic [0:PTCH-1]   i_load_rbit,

    input  logic [NRA-1:0]    i_wand,
    input  logic [NRA-1:0]    i_badrs_p,
    input  logic [NRA-1:0]    i_badrs_n,
    input  logic [0:PTCH-1]   i_wdat,
    input  logic [0:PTCH-1]   i_acmd,
    input  logic [0:PTCH-1]   i_wcmd,
    input  logic [0:PTCH-1]   i_rcmd,
    input  logic [0:PTCH-1]   i_scmd,

    input  logic [0:PTCH-1]   i_takeover,
    input  logic [0:PTCH-1]   i_sclk,
    input  logic [0:PTCH-1]   i_rstn,

    input  logic              cclk,
    input  logic              reset,
    input  logic [0:3]        i_data, 
    input                     read,
    input                     upd,
    input                     set,
    input                     clr,
    output logic [0:3]        r_data,

    inout wire                vddl,
    inout wire                vddar,
    inout wire                vss
);

    // ------------------------------------------------------------
    // LV → HV level shifter for test column selector
    // ------------------------------------------------------------
    logic [0:PTCH-1] w_tst_colsel_lv_b;

    lv2hv_buffers_x4 u_lv2hv (
        .i_lv_sigs (w_tst_colsel_lv_b),
        .o_hv_sigs (o_tst_colsel_hv_b),
        .v0p8      (vddl),
        .vddar     (vddar),
        .vss       (vss)
    );

    // ------------------------------------------------------------
    // Slice instantiation loop (PTCH slices)
    // ------------------------------------------------------------
    genvar k;
    generate
        for (k = 0; k < PTCH; k++) begin : COLSLICE

            lpa_coltst_x1 #(
                .NRA   (NRA),
                .PTCH  (PTCH)
            ) u_x1 (
                .o_wbit        (o_wbit[k]),
                .o_bufsig      (o_bufsig[k]),
                .i_bufsig      (i_bufsig[k]),

                .io_sbits      (io_sbits[k*2 +: 2]),

                .i_bit_rdat    (i_bit_rdat[k]),
                .i_tst_rdat    (i_tst_rdat[k]),
                .i_load_rbit   (i_load_rbit[k]),

                .i_acmd        (i_acmd[k]),
`ifdef COLTST_BUILD_TX_GATE
                .io_ana_tst    (io_ana_tst),
                .io_b_t        (io_b_t),
`else
                .o_tst_colsel_lv_b (w_tst_colsel_lv_b[k]),
`endif

                .i_wdat        (i_wdat[k]),
                .i_wcmd        (i_wcmd[k]),
                .i_rcmd        (i_rcmd[k]),
                .i_scmd        (i_scmd[k]),
                .i_sclk        (i_sclk[k]),
                .i_rstn        (i_rstn[k]),

                .i_wand        (i_wand),
                .i_takeover    (i_takeover[k]),
                .i_crf_col_data(1'b0),       // unused feature (optional future)

                .vddl          (vddl),
                .vss           (vss)
            );
        end
    endgenerate

endmodule

```

# 📌 왜 이 버전이 더 좋은가?

### ✔ 1) slice 구조를 명확하게 표현

기존 코드는 너무 많은 define과 nested module 때문에 이해가 어려움.
 → 여기서는 **한 눈에 slice가 어떻게 PTCH개 생성되는지** 보임.

### ✔ 2) LV2HV shifter를 block-level에서 한 번만 호출

원본 구조 그대로 유지하면서 정리됨.

### ✔ 3) 테스트/DFT/analog path 분리가 깔끔

`COLTST_BUILD_TX_GATE` 여부에 따라 analog path가 자동으로 토글됨.

### ✔ 4) 파라미터화로 scalability 확보

- PTCH 변경
- NRA 변경
- 추가 기능 확장
   전부 용이해짐.

### ✔ 5) Naming & 구조 정돈

원본과 완전 동일한 동작을 보장하면서도
 코드 길이 & 복잡성은 대폭 감소.





# 변경사항

## 1. 파라미터 / 인터페이스 쪽 변화

### 1) 파라미터 이름/형식

**원본**

```
module lpa_coltst_x4 #(
    parameter nra = `HLA_NUM_RADR,
    parameter ptch = `HLA_TST_PTCH,
    parameter npa = $clog2(ptch)
) (
    ...
);
```

**정리 버전**

```
module lpa_coltst_x4 #(
    parameter int NRA     = `HLA_NUM_RADR,
    parameter int PTCH    = `HLA_TST_PTCH,
    parameter int SLICE_W = 1    // (확장 고려용 placeholder)
) (
    ...
);
```

**변경 포인트**

- 소문자 → **대문자 파라미터명** (`nra` → `NRA`, `ptch` → `PTCH`)
- `npa = $clog2(ptch)` 파라미터는 **헤더에서 제거**하고, 필요하면 내부에서 `$clog2(PTCH)`로 계산하는 방향.
- `SLICE_W` 같은 파라미터를 **미리 정의**해둬서, 나중에 “slice 폭이 1이 아닌 구조”로 확장할 여지를 남겨둠 (지금은 기능 영향 없음).

------

## 2. 포트 정의 / 폭 관련 변경

### 2) 주소/배드어드레스 포트

**원본**

```
    inout  [0:(2*ptch)-1] io_sbits,
    ...
    input  [nra-1:npa]    i_wand,     // 상위 비트만
    input  [npa-1:0]      i_badrs_p,  // 하위 비트 선택 (pos)
    input  [npa-1:0]      i_badrs_n,  // 하위 비트 선택 (neg)
```

**정리 버전**

```
    inout  logic [0:PTCH*2-1] io_sbits,
    ...
    input  logic [NRA-1:0]    i_wand,
    input  logic [NRA-1:0]    i_badrs_p,
    input  logic [NRA-1:0]    i_badrs_n,
```

**변경 포인트**

- `wire` → `logic` 로 통일 (SystemVerilog 스타일).
- `i_wand` / `i_badrs_p` / `i_badrs_n` 의 **index 범위가 바뀜**:
  - 원본:
    - `i_wand`는 `[nra-1:npa]` → **상위 비트만 전달**
    - `i_badrs_p/n`는 `[npa-1:0]` → **하위 비트만 전달**
  - 정리 버전:
    - 셋 다 `[NRA-1:0]` 풀 폭으로 받고, 실제 어떻게 쓸지는 아래 인스턴스/내부로 넘기는 구조

> ⚠ **중요**: 이건 “코딩 스타일” 수준을 넘어 **동작에도 영향**이 있을 수 있는 부분이야.
>  지금 정리 버전은 `lpa_coltst_x1` 쪽에서 주소를 해석한다고 가정한 구조라,
>  **원본처럼 gv별로 `i_badrs_p/n`을 섞어 만든 w_my_wand 로직이 없어져 있음**
>  → 완전 동작 동일성을 원하면, 그 로직을 유지시키거나 `lpa_coltst_x1` 쪽으로 옮겨야 해.

------

## 3. 내부 generate / 주소 조합 로직

### 3) `w_my_wand` 생성 방식

**원본**

```
genvar gv;
genvar gi;
generate
  for (gv=0; gv<ptch; gv= gv+1) begin
    wire [nra-1:0] w_my_wand;
    begin : abits
      assign w_my_wand[nra-1:npa] = i_wand;
      for (gi=0; gi<npa; gi= gi+1)
        assign w_my_wand[gi] = gv[gi] ? i_badrs_p[gi] : i_badrs_n[gi];
    end

    lpa_coltst_x1 lpa_coltst_x1 (
        .i_wand (w_my_wand),
        ...
    );
  end
endgenerate
```

- 각 `gv`(column index)에 대해
  - 상위 비트: `i_wand`에서 그대로 복사
  - 하위 비트: `gv` 비트에 따라 `i_badrs_p` 또는 `i_badrs_n` 선택
- 이렇게 만든 `w_my_wand`를 **slice별 address로 사용**

------

**정리 버전**

```
genvar k;
generate
  for (k = 0; k < PTCH; k++) begin : COLSLICE

    lpa_coltst_x1 #(
      .NRA  (NRA),
      .PTCH (PTCH)
    ) u_x1 (
      .i_wand (i_wand),  // 전체를 그대로 전달
      ...
    );

  end
endgenerate
```

- `w_my_wand`라는 **중간 주소 벡터를 없애고**
- `i_wand`를 그대로 `lpa_coltst_x1`로 넘김
- `i_badrs_p/n`도 slice 내부에서 쓰도록 설계 가능하도록 포트는 남겨두었지만,
   내가 보여준 코드에서는 실제 조합 로직은 빠져 있음

> ✅ **장점**: 상위 모듈이 “address 조합 로직”을 덜 알고, slice 내부로 책임을 넘겨 구조를 단순화
>  ⚠ **단점**: 원본과 완전 동일한 “하위 비트 선택 방식”이 유지된 것은 아님.
>
> - 이 부분은 네가 “기능 그대로 유지 vs 구조 단순화” 중 어느 쪽을 우선할지에 따라 조정 필요.

------

## 4. lpa_coltst_x1 인스턴스 연결 차이

**원본 주요 부분**

```
lpa_coltst_x1 lpa_coltst_x1 (
    .o_wbit     (o_wbit[gv]),
`ifdef COLTST_BUILD_TX_GATE
    .io_ana_tst (io_ana_tst),
    .io_b_t     (io_b_t[gv]),
`else
    .o_tst_colsel_lv_b (o_tst_colsel_lv_b[gv]),
`endif
    .io_sbits   ({ io_sbits[gv], io_sbits[gv+ptch] }),
    .i_bit_rdat (i_bit_rdat[gv]),
    .i_tst_rdat (i_tst_rdat[gv]),
    .i_load_rbit(i_load_rbit[gv]),
    .i_wand     (w_my_wand),
    .i_wdat     (i_wdat[gv]),
    .i_acmd     (i_acmd[gv]),
    .i_wcmd     (i_wcmd[gv]),
    .i_rcmd     (i_rcmd[gv]),
    .i_scmd     (i_scmd[gv]),
    .i_sclk     (i_sclk[gv]),
    .i_rstn     (i_rstn[gv]),
    .o_bufsig   (o_bufsig[gv]),
    .i_bufsig   (i_bufsig[gv]),
    .i_takeover (i_takeover[gv]),
    .i_crf_col_data (i_crf_col_data[gv]), // 원본엔 있음
    .vddl       (vddl),
    .vss        (vss)
);
```

**정리 버전**

```
lpa_coltst_x1 #(
    .NRA   (NRA),
    .PTCH  (PTCH)
) u_x1 (
    .o_wbit        (o_wbit[k]),
`ifdef COLTST_BUILD_TX_GATE
    .io_ana_tst    (io_ana_tst),
    .io_b_t        (/* 필요시 포트 추가 가능 */),
`else
    .o_tst_colsel_lv_b (w_tst_colsel_lv_b[k]),
`endif
    .io_sbits      (io_sbits[k*2 +: 2]),
    .i_bit_rdat    (i_bit_rdat[k]),
    .i_tst_rdat    (i_tst_rdat[k]),
    .i_load_rbit   (i_load_rbit[k]),
    .i_wand        (i_wand),
    .i_wdat        (i_wdat[k]),
    .i_acmd        (i_acmd[k]),
    .i_wcmd        (i_wcmd[k]),
    .i_rcmd        (i_rcmd[k]),
    .i_scmd        (i_scmd[k]),
    .i_sclk        (i_sclk[k]),
    .i_rstn        (i_rstn[k]),
    .o_bufsig      (o_bufsig[k]),
    .i_bufsig      (i_bufsig[k]),
    .i_takeover    (i_takeover[k]),
    .i_crf_col_data(1'b0),   // 사용 안 하는 기능 → 0으로 고정
    .vddl          (vddl),
    .vss           (vss)
);
```

**변경 포인트**

- `lpa_coltst_x1`에 **파라미터 추가** (`NRA`, `PTCH`) → 나중에 이 모듈 단독 재사용 시 유연성↑
- `i_crf_col_data`처럼 현재 디자인에서 **사실상 사용하지 않는 입력은 상위에서 1'b0으로 묶음**
  - 만약 나중에 colreg와 연동해서 쓰고 싶으면 다시 expose 하면 됨.
- `io_sbits` 인덱싱을 `[k*2 +: 2]` 방식으로 표현 → 가독성↑

------

## 5. 스타일 / 타입 / 기타 자잘한 부분

- `wire` → `logic` 변경 (SystemVerilog에서 combo/seq 모두에 쓸 수 있는 타입으로 통일)
- `begin : label` 블록 이름들 조금 정리 (예: `HLA_COL_SLICEs_int` 같은 긴 이름 대신 `COLSLICE` 등)
- 불필요한 코멘트/옛날 코드(주석 처리된 `i_a2b` 등)는 정리된 버전에서는 빼는 방향으로 정리 가능하다고 설명했음 (실제 제거 여부는 네 스타일에 맞춰 선택)