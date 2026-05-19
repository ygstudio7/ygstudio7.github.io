



| **모드**      | **odat_ne (clk=0)** | **odat_pe (clk=1)** | **비고**                  |
| ------------- | ------------------- | ------------------- | ------------------------- |
| **SDR PHA=0** | `tsft[7:4]`         | `tsft_pe[7:4]`      | 표준 SPI, 안정적인 출력   |
| **SDR PHA=1** | `tsft_pe[7:4]`      | `tsft_pe[7:4]`      | `posedge`에서 변하고 유지 |
| **DDR PHA=0** | `tsft[7:4]`         | `tsft_pe[3:0]`      | `ne`에서 상위비트 시작    |
| **DDR PHA=1** | `tsft_pe[3:0]`      | `tsft_pe[7:4]`      | `pe`에서 상위비트 시작    |





```verilog
//----------------------------------------------------------------------------
// Tx data Output Logic (Final Optimized & Unified)
//----------------------------------------------------------------------------

  // 1. tsft_pe: posedge clk에서 tsft를 그대로 래치 (shift 없음)
  // 모든 모드에서 clk=1(High) 구간의 데이터 안정성을 보장하는 소스로 사용됨
  logic [DW-1:0] tsft_pe;
  always_ff @(posedge clk or posedge csn) begin
    if(csn) tsft_pe <= '0;
    else    tsft_pe <= tsft; 
  end

  // 2. Mux 입력 데이터 할당 ({odat_ne, odat_pe})
  // odat_ne: clk=0 (Low) 구간 데이터
  // odat_pe: clk=1 (High) 구간 데이터
  always_comb begin
    odat_ne = '0;
    odat_pe = '0;

    // --- Case 1: DDR 모드 ---
    if (cfg.ddr) begin
      if (cfg.pha) begin
        // DDR + PHA=1: posedge에서 상위비트 시작(pe), negedge에서 하위비트 전환(ne)
        // negedge에서 tsft가 이미 shift되었으므로, ne구간은 tsft_pe의 하위비트 유지
        case(cfg.wid)
          2'd0:    {odat_ne, odat_pe} = {{3'b0, tsft_pe[DW-2]},    {3'b0, tsft_pe[DW-1]}};
          2'd1:    {odat_ne, odat_pe} = {{2'b0, tsft_pe[DW-3-:2]}, {2'b0, tsft_pe[DW-1-:2]}};
          default: {odat_ne, odat_pe} = {tsft_pe[DW-5-:4],         tsft_pe[DW-1-:4]};
        endcase
      end 
      else begin
        // DDR + PHA=0: negedge에서 상위비트 시작(ne), posedge에서 하위비트 전환(pe)
        case(cfg.wid)
          2'd0:    {odat_ne, odat_pe} = {{3'b0, tsft[DW-1]},       {3'b0, tsft_pe[DW-2]}};
          2'd1:    {odat_ne, odat_pe} = {{2'b0, tsft[DW-1-:2]},    {2'b0, tsft_pe[DW-3-:2]}};
          default: {odat_ne, odat_pe} = {tsft[DW-1-:4],            tsft_pe[DW-5-:4]};
        endcase
      end
    end 

    // --- Case 2: SDR 모드 + PHA=1 ---
    else if (cfg.pha) begin
      // Leading Edge(pe)에서 데이터 변화 시작.
      // 다음 posedge 전까지 데이터를 유지하기 위해 ne, pe 모두 tsft_pe 참조
      case(cfg.wid)
        2'd0:    {odat_ne, odat_pe} = {{3'b0, tsft_pe[DW-1]},    {3'b0, tsft_pe[DW-1]}};
        2'd1:    {odat_ne, odat_pe} = {{2'b0, tsft_pe[DW-1-:2]}, {2'b0, tsft_pe[DW-1-:2]}};
        default: {odat_ne, odat_pe} = {tsft_pe[DW-1-:4],         tsft_pe[DW-1-:4]};
      endcase
    end

    // --- Case 3: SDR 모드 + PHA=0 ---
    else begin
      // Trailing Edge(ne)에서 데이터 업데이트.
      // pe구간은 posedge에서 latch된 tsft_pe를 사용하여 출력 안정성 강화
      case(cfg.wid)
        2'd0:    {odat_ne, odat_pe} = {{3'b0, tsft[DW-1]},       {3'b0, tsft_pe[DW-1]}};
        2'd1:    {odat_ne, odat_pe} = {{2'b0, tsft[DW-1-:2]},    {2'b0, tsft_pe[DW-1-:2]}};
        default: {odat_ne, odat_pe} = {tsft[DW-1-:4],            tsft_pe[DW-1-:4]};
      endcase
    end
  end

  // 3. 최종 출력 Mux
  // clk=0일 때 odat_ne, clk=1일 때 odat_pe 출력
  gpl_mux2 spi_ddr_mux [3:0] (
    .sel    (clk),
    .in0    (odat_ne),
    .in1    (odat_pe),
    .out    (odat)
  );
```

더 최적화한 코드

```verilog
//----------------------------------------------------------------------------
// Tx data Output Logic (Refined Naming & Optimized)
//----------------------------------------------------------------------------

  // 1. Posedge Snapshot
  logic [DW-1:0] tsft_pe;
  always_ff @(posedge clk or posedge csn) begin
    if(csn) tsft_pe <= '0;
    else    tsft_pe <= tsft; 
  end

  // 2. 비트 그룹 정의 (h: 상위 뭉치, l: 하위 뭉치)
  logic [3:0] tsft_h, tsft_l;
  always_comb begin
    case(cfg.wid)
      2'd0:    begin tsft_h = {3'b0, tsft[DW-1]};    tsft_l = {3'b0, tsft[DW-2]};    end
      2'd1:    begin tsft_h = {2'b0, tsft[DW-1-:2]}; tsft_l = {2'b0, tsft[DW-3-:2]}; end
      default: begin tsft_h = tsft[DW-1-:4];         tsft_l = tsft[DW-5-:4];         end
    endcase
  end

  // 래치된 버전 (pe 기반 슬라이싱)
  logic [3:0] tsft_pe_h, tsft_pe_l;
  assign tsft_pe_h = (cfg.wid == 2'd0) ? {3'b0, tsft_pe[DW-1]}    : (cfg.wid == 2'd1) ? {2'b0, tsft_pe[DW-1-:2]} : tsft_pe[DW-1-:4];
  assign tsft_pe_l = (cfg.wid == 2'd0) ? {3'b0, tsft_pe[DW-2]}    : (cfg.wid == 2'd1) ? {2'b0, tsft_pe[DW-3-:2]} : tsft_pe[DW-5-:4];

  // 3. 최종 출력 선택 로직
  always_comb begin
    if (cfg.ddr) begin
      if (cfg.pha) begin
        // DDR PHA=1: pe(High비트 시작) -> ne(Low비트로 전환)
        odat_pe = tsft_pe_h; 
        odat_ne = tsft_pe_l; 
      end else begin
        // DDR PHA=0: ne(High비트 시작) -> pe(Low비트로 전환)
        odat_ne = tsft_h;    
        odat_pe = tsft_pe_l; 
      end
    end 
    else begin // SDR 모드
      if (cfg.pha) begin
        // SDR PHA=1: 한 주기 내내 pe에서 래치된 High비트 유지
        odat_pe = tsft_pe_h;
        odat_ne = tsft_pe_h;
      end else begin
        // SDR PHA=0: ne(tsft)에서 pe(tsft_pe)까지 High비트 유지
        odat_ne = tsft_h;
        odat_pe = tsft_pe_h;
      end
    end
  end

  // 4. 최종 출력 Mux
  gpl_mux2 spi_ddr_mux [3:0] (
    .sel(clk), 
    .in0(odat_ne), 
    .in1(odat_pe), 
    .out(odat)
  );
```

주석 영어로

```verilog
//----------------------------------------------------------------------------
// Tx Data Output Logic (Optimized for SDR/DDR and PHA 0/1)
//----------------------------------------------------------------------------

  // 1. Posedge Snapshot (Latching tsft at posedge clk)
  // Used as a stable data source for clk=1 (High) intervals in all modes
  logic [DW-1:0] tsft_pe;
  always_ff @(posedge clk or posedge csn) begin
    if(csn) tsft_pe <= '0;
    else    tsft_pe <= tsft; 
  end

  // 2. Data Grouping (h: High/MSB part, l: Low/LSB part)
  logic [3:0] tsft_h, tsft_l;
  always_comb begin
    case(cfg.wid)
      2'd0:    begin tsft_h = {3'b0, tsft[DW-1]};    tsft_l = {3'b0, tsft[DW-2]};    end
      2'd1:    begin tsft_h = {2'b0, tsft[DW-1-:2]}; tsft_l = {2'b0, tsft[DW-3-:2]}; end
      default: begin tsft_h = tsft[DW-1-:4];         tsft_l = tsft[DW-5-:4];         end
    endcase
  end

  // Latched data grouping based on tsft_pe
  logic [3:0] tsft_pe_h, tsft_pe_l;
  assign tsft_pe_h = (cfg.wid == 2'd0) ? {3'b0, tsft_pe[DW-1]}    : (cfg.wid == 2'd1) ? {2'b0, tsft_pe[DW-1-:2]} : tsft_pe[DW-1-:4];
  assign tsft_pe_l = (cfg.wid == 2'd0) ? {3'b0, tsft_pe[DW-2]}    : (cfg.wid == 2'd1) ? {2'b0, tsft_pe[DW-3-:2]} : tsft_pe[DW-5-:4];

  // 3. Final Output Selection (odat_ne for clk=0, odat_pe for clk=1)
  always_comb begin
    if (cfg.ddr) begin
      if (cfg.pha) begin
        // DDR PHA=1: Start with MSB at posedge(pe) -> Switch to LSB at negedge(ne)
        odat_pe = tsft_pe_h; 
        odat_ne = tsft_pe_l; 
      end else begin
        // DDR PHA=0: Start with MSB at negedge(ne) -> Switch to LSB at posedge(pe)
        odat_ne = tsft_h;    
        odat_pe = tsft_pe_l; 
      end
    end 
    else begin // SDR Mode
      if (cfg.pha) begin
        // SDR PHA=1: Hold latched MSB through the entire cycle (starts at posedge)
        odat_pe = tsft_pe_h;
        odat_ne = tsft_pe_h;
      end else begin
        // SDR PHA=0: Hold MSB through the entire cycle (starts at negedge)
        odat_ne = tsft_h;
        odat_pe = tsft_pe_h;
      end
    end
  end

  // 4. Final Output Mux (ODDR structure)
  // sel=0 (Low): outputs odat_ne, sel=1 (High): outputs odat_pe
  gpl_mux2 spi_ddr_mux [3:0] (
    .sel(clk), 
    .in0(odat_ne), 
    .in1(odat_pe), 
    .out(odat)
  );
```



