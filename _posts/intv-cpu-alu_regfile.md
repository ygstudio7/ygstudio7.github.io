# 19) ALU / Register File / Memory Controller

- **ALU**: 단일사이클 vs 파이프라인(주파수↑), 플래그(Zero/Carry/Overflow) 정의, 멀티플라이/디바이더 선택(면적/지연 trade-off).
- **RegFile**: 포트 수(PNR 영향 큼), true/false dual-port RAM 선택, **FWP(Forwarding)** 또는 **bypass**로 hazard 완화.
- **Mem Ctrl**: 타이밍(DDR tRCD/tRP 등) 스케줄러, QoS/arbiter(가중치/나이), write-leveling, ECC.



# 20) 캐시(L1/L2/L3) & 교체정책

- **L1**(작고 빠름), **L2**(중간), **L3**(크고 느림, 공유).
- **교체**: LRU, PLRU, Random, SRRIP. 워킹-셋/스레드 간 간섭 고려.
- **면접 포인트**: *히트율·지연·면적* 간 최적화 시나리오 제시.

`TODO` L3 cache의 공유방식

# 23) Skid Buffer (1-딥 버블-프리)

- **개념**: `ready`가 늦게 떨어져도 **마지막 전송 데이터 1개**를 안전 보관 → 버블 없이 1 beat/clk 유지.
- 

```verilog
module skid #(parameter W=64)(
  input  logic         clk, rstn,
  input  logic [W-1:0] s_data,
  input  logic         s_valid,
  output logic         s_ready,
  output logic [W-1:0] m_data,
  output logic         m_valid,
  input  logic         m_ready
);
  logic [W-1:0] hold;
  logic         hold_v;

  // Upstream ready: downstream ready or buffer empty
  assign s_ready = m_ready || !hold_v;

  // MUX bypass when no hold
  assign m_valid = hold_v ? 1'b1 : s_valid;
  assign m_data  = hold_v ? hold  : s_data;

  always_ff @(posedge clk or negedge rstn) begin
    if(!rstn) hold_v <= 1'b0;
    else begin
      // capture when upstream fires but downstream can't
      if (s_valid && s_ready && !m_ready) begin
        hold   <= s_data;
        hold_v <= 1'b1;
      end else if (m_ready && hold_v) begin
        hold_v <= 1'b0;
      end
    end
  end
endmodule
```

- **포인트**: `ready`의 콤비 경로를 짧게, 타이밍 여유 ↑.