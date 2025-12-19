History:

- 251201: Revised



# Part 1. Basic



# ✅ 1) Reset (Async vs Sync)

## ✔ 정의

- **Async Reset**
  - 클록과 무관하게 즉시 reset assert/deassert
  - 일반적으로 *assert는 빠름*, *deassert는 위험함*
- **Sync Reset**
  - 클록 엣지에서만 reset assert/deassert
  - 항상 clock domain 내에서 안전하게 동작

------

# ✔ 언제/왜 사용?

### 🔹 Async Reset – 장점/사용 이유

- **Power-up 직후 clock이 나오기 전에 반드시 초기화가 필요할 때**
- 대부분의 ASIC/FPGA 라이브러리에서 async reset pin이 존재 → 초기에 확실한 초기화 가능

### 🔹 Sync Reset – 장점/사용 이유

- **Reset release가 clock에 정합되어 안전**
   → 글리치, meta, recovery/removal 문제 감소
- **타이밍 분석이 단순해짐**
   → reset을 일반 데이터처럼 STA 가능
- **Clock gating/LPI 블록에서 우수**
   (gated clock이면 async reset이 블록 안으로 전달되지 않을 수 있음)

------

# ✔ 주의점

### 🔹 Async Reset 단점

- **Deassert 타이밍 위험(recovery/removal violation)**
- **각 flip-flop에 reset net delay가 달라 불일치 상태로 시작할 수 있음**
- **release 시 반드시 2-stage synchronizer 필요**

### 🔹 Sync Reset 단점

- Clock이 살아있어야 reset이 풀림 (clock이 죽어 있으면 reset이 의미 없음)

------

# ⭐ 결론 한 줄

> **"Async assert는 괜찮지만, release는 반드시 clock에 sync해서 풀어라."**
>  (업계 표준 규칙)

------

# 🚀 Q1) Why must async reset be *sync-released*?

async reset을 “동기화해서 풀어야 하는 이유”는 아래 3가지가 핵심입니다.

------

## ① Reset deassert 시 metastability 위험

- assert는 문제 없음 (clock 무시하고 무조건 초기화)
- **deassert는 clock edge 근처에서 일어나면 meta 발생**
- → FF마다 meta 상태가 생기면 register/FSM이 **불확정 상태**로 시작할 수 있음

------

## ② Reset release skew → 설계 전체가 “다른 cycle”에서 시작

- 하나의 async reset net이 huge fanout → wiring delay가 각 FF마다 다름
- reset release가 FF마다 **서로 다른 clock 주기**에서 발생할 수 있음
- 결과:
  - FSM state mismatch
  - pipeline stage misalignment
  - multi-register state machine에서 catastrophic bug 발생

→ 이 문제 때문에 ASIC에서는 **거의 강제적으로 sync release**를 넣음.

------

## ③ STA 관점에서 async deassert는 타이밍 처리 복잡

- async release는 **Recovery/Removal** 체크 필요
- reset net skew가 커지면 violation 발생하기 매우 쉬움
- 반면 sync release는:
  - reset release를 **FF → sync FF → logic** 형태로 STA가 일반 data path로 취급 가능
  - 타이밍 분석 **너무 쉬워짐**
- 그래서 **EDA tool이 sync release를 강력 추천**
   (또는 자동으로 CDC rule에서 warning 띄움) 

------



# 👍 면접/리뷰용 최종 답변 요약 (3줄)

> **1. reset release가 clock 경계 근처면 FF가 metastability에 빠진다.**
>  **2. <u>reset skew 때문에 FF들이 서로 다른 cycle에서 reset 해제</u>되어 시스템이 불일치 상태로 시작한다.**
>  **3. async release는 recovery/removal 때문에 STA가 매우 복잡하고 violation이 쉽게 난다. sync release는 이를 완전히 해결한다.**



# Part 2. Practice



# ✅ (A) Reset Synchronizer – ASIC Best Practice Code

### ✔ 2FF Sync Release (가장 표준)

```
module reset_sync (
    input  logic clk,
    input  logic rst_n_async,   // async assert, async deassert (external reset)
    output logic rst_n_sync     // async assert, sync deassert
);

    logic [1:0] sync_ff;

    // Async assert, sync deassert
    always_ff @(posedge clk or negedge rst_n_async) begin
        if (!rst_n_async) begin
            sync_ff <= 2'b00;
        end else begin
            sync_ff <= {sync_ff[0], 1'b1};
        end
    end

    assign rst_n_sync = sync_ff[1];

endmodule
```

### ✔ 특징

- Assert는 즉시 전달 (`rst_n_async` low)
- Deassert는 FF 두 번 통과 → metastability filtering
- 전체 FSM/Register들은 **rst_n_sync** 사용

------

# ✅ (B) ASIC/SoC에서 Reset 연결 구조

```
external_reset -----> reset_sync -----> block_reset (사용)
                          |
                          | (2FF Sync)
                          v
                    safe reset tree
```

ASIC에서는 reset이 기본적으로 huge fanout이 되어 CTS 수준으로 처리해야 하므로,
 **sync release 후에 reset distribution**이 이뤄져야 한다.

------

# ✅ (C) ASIC vs FPGA Reset 전략 차이

| 구분                     | ASIC                                      | FPGA                                             |
| ------------------------ | ----------------------------------------- | ------------------------------------------------ |
| Power-up 초기화          | 반드시 async reset 필요                   | 대부분 FPGA는 내부 LUT FF가 power-up 초기화 지원 |
| Reset distribution       | 뒤틀림(skew) 매우 큼 → sync release 필수  | FPGA fabric routing 규칙적으로 배치됨            |
| Clock gating 많은 SoC    | sync reset이 매우 유리                    | FPGA에서는 종종 local reset fanout 허용          |
| Reset이 타이밍 critical? | recovery/removal 위협 → sync release 필수 | FPGA는 tool이 자동 해결 많은 편                  |

**즉:** ASIC은 거의 무조건 *assert async, release sync* 구조 사용.
 FPGA는 간단한 디자인이면 sync-only reset도 자주 씀.

------

# ✅ (D) 면접용 1분 답변 (완성형)

> “Async reset은 assert는 빠르고 좋지만, release가 clock-edge 근처면 FF가 metastability에 빠질 수 있습니다.
>  그리고 reset net skew로 인해 FF마다 서로 다른 clock cycle에서 reset이 풀리면 FSM이 불일치 상태에서 시작하게 됩니다.
>  마지막으로 async release는 recovery/removal 타이밍 분석이 복잡하기 때문에 STA 부담이 큽니다.
>  그래서 업계 표준은 **async assert, sync release** 방식이며 2FF synchronizer로 reset release를 항상 clock에 정렬시키는 것입니다.”

30초로 줄여줄 수도 있음.

------

# ✅ (E) Clock Gating 환경에서 Reset Best Practice

Clock gating이 많아질수록 아래 두 가지가 중요:

### ✔ 1) Gated clock domain reset은 반드시 sync release

gclk가 low에서 stuck된 상태로 reset이 풀리면 block이 살아나지 못함.

### ✔ 2) Reset은 “ungated clock domain에서 sync 후” gating 내부로 분배

```
clk ----> gating ----> gclk ---> FFs
  \
   \__ reset_sync (clk domain)
```

### ✔ 3) gating enable은 reset 동안 강제로 open

```
assign g_en_safe = g_en | ~rst_n_sync;
```

reset 동안 clock이 들어가야 safe state로 초기화됨.







# Part 3. 실무

# 🔥 (F) 당신 프로젝트 상황에 맞춰 custom version도 만들어드릴 수 있어요

예를 들어:

- SoC top에서 reset tree 3-level hierarchy
- A프론트엔드 ISP pipeline(flow delay large) reset design pattern
- clock domain crossing이 많은 구조 (multi-PE systolic array)
- FPGA prototype에서 async reset을 sync-only reset으로 대체하는 법
- Synopsys Design Compiler · PrimeTime · Innovus 기준 best practice
- Questa vs VCS에서 recovery/removal 메시지 억제 또는 분석법

원하시는 방향 알려주면 바로 맞춰서 만들어 드릴게요.

##### You said:

모두

###### ChatGPT said:

좋아요 😄
 그럼 아까 말한 것들 **전부** 한 번에 정리해서,
 실제 네가 쓰는 **ASIC/SoC + ISP + systolic + FPGA proto + Synopsys/Questa/VCS** 전부 커버하는 “리셋 설계 패키지”로 만들어볼게.

------

## 0. 공통: 표준 Reset Sync 모듈 (Async assert / Sync deassert)

이거 하나를 **모든 clock domain마다** 둔다고 생각하면 됨.

```
// Async assert, Sync deassert reset synchronizer
module reset_sync #(
    parameter STAGES = 2  // 일반적으로 2, 필요하면 3
) (
    input  logic clk,
    input  logic rst_n_async,   // 외부에서 들어오는 async reset
    output logic rst_n_sync     // 블록 내부에서 쓰는 sync reset
);

    logic [STAGES-1:0] sync_ff;

    always_ff @(posedge clk or negedge rst_n_async) begin
        if (!rst_n_async) begin
            sync_ff <= '0;
        end else begin
            sync_ff <= {sync_ff[STAGES-2:0], 1'b1};
        end
    end

    assign rst_n_sync = sync_ff[STAGES-1];

endmodule
```

> 룰: **외부/상위 reset = rst_n_async**,
>  **블록/도메인 내부는 항상 rst_n_sync만 사용**.

------

## 1. SoC Top Reset Tree – 3레벨 계층 예시

### 1-1. 개념 구조

보통 SoC reset은 이런 식:

1. **Level 0 (Global POR)**
   - `por_n`: power-on reset, analog POR, external pin, watchdog, etc.
2. **Level 1 (Mode별 reset)**
   - `cold_rst_n`, `warm_rst_n`, `scan_rst_n` 같은 것들
3. **Level 2 (각 clock domain / block별 reset)**
   - `rst_n_cpu`, `rst_n_ddr`, `rst_n_isp`, `rst_n_noc` …

ASCII로 그리면:

```
            por_n (external / analog POR)
                     |
           +---------+-----------+
           |                     |
      cold_rst_n            warm_rst_n
           |                     |
   +-------+------+             ...
   |              |
 rst_n_cpu    rst_n_isp   (각 domain별)
   |              |
 reset_sync    reset_sync
 (cpu_clk)     (isp_clk)
   |              |
 cpu blk       isp blk
```

### 1-2. SoC Top 예제 코드 (간단한 형태)

```
module soc_top (
    input  logic por_n,        // global async reset
    input  logic cpu_clk,
    input  logic isp_clk
);

    // Level1: cold reset (여기선 por_n 그대로 사용했다고 가정)
    logic cold_rst_n = por_n;

    // Domain별 sync reset
    logic cpu_rst_n_sync;
    logic isp_rst_n_sync;

    reset_sync u_cpu_rst_sync (
        .clk         (cpu_clk),
        .rst_n_async (cold_rst_n),
        .rst_n_sync  (cpu_rst_n_sync)
    );

    reset_sync u_isp_rst_sync (
        .clk         (isp_clk),
        .rst_n_async (cold_rst_n),
        .rst_n_sync  (isp_rst_n_sync)
    );

    cpu_subsystem u_cpu (
        .clk    (cpu_clk),
        .rst_n  (cpu_rst_n_sync)
    );

    isp_top u_isp (
        .clk    (isp_clk),
        .rst_n  (isp_rst_n_sync)
    );

endmodule
```

> 포인트: **외부 reset은 절대 직접 block에 꽂지 말고**,
>  항상 중간에 `reset_sync`를 거쳐서 domain별로 clean하게 쓴다.

------

## 2. ISP / Imaging Pipeline Reset 패턴

네가 하고 있는 ISP/이미징 파이프라인 같이 **stage 수가 많고, 데이터폭 큰 파이프라인**에서는:

### 2-1. “모든 data flop을 reset” → 비효율

- 수백~수천 개의 data register를 전부 reset =
  - area 증가, routing 복잡, reset tree 부담
- 보통 **data path는 reset 안 하고**,
   **control/valid/state만 reset**하는 구조로 가는 게 일반적.

### 2-2. 실무 패턴 – `valid` 기반 flush

```
typedef struct packed {
    logic [DATA_W-1:0] pix;
    logic              valid;
} pix_bus_t;

module isp_stage (
    input  logic     clk,
    input  logic     rst_n,      // sync reset
    input  pix_bus_t in,
    output pix_bus_t out
);

    pix_bus_t r;

    always_ff @(posedge clk) begin
        if (!rst_n) begin
            r.valid <= 1'b0;    // control만 reset
            // r.pix는 reset 안 함 (don’t care)
        end else begin
            r <= in;
        end
    end

    assign out = r;

endmodule
```

파이프라인 전체도:

- **상위에서 전체 `valid`를 0으로 초기화**
- reset 이후 일정 cycle 동안 output은 invalid (0) → consumer가 무시

→ ISP 같은 continuous stream 디자인에서 **가장 깔끔하고 저비용**.

### 2-3. FSM/Config 레지스터는 반드시 reset

- FSM state
- mode/config register (e.g., filter enable, scaling factor 등)
- frame boundary control (SOF/EOF detector)

이런 부분은 **동작이 deterministic하지 않으면 debugging 지옥**이므로 반드시 sync reset으로 초기화.

------

## 3. Multi-PE Systolic Array & CDC 많은 구조에서 Reset

네가 하는 **multi-bank systolic (3×2 PE 등)**은:

- 여러 clock domain (ex: core_clk, noc_clk, ddr_clk…)
- CDC FIFO, handshake, AXI interface 등 많음

### 3-1. 도메인별 reset sync + reset sequencing

각 domain마다:

```
reset_sync u_core_rst_sync (.clk(core_clk), .rst_n_async(cold_rst_n), .rst_n_sync(core_rst_n));
reset_sync u_noc_rst_sync  (.clk(noc_clk ), .rst_n_async(cold_rst_n), .rst_n_sync(noc_rst_n ));
reset_sync u_ddr_rst_sync  (.clk(ddr_clk ), .rst_n_async(cold_rst_n), .rst_n_sync(ddr_rst_n ));
```

**추가로 중요한 것**:

- domain 간 **reset release 순서**를 설계할 것
  - 예: DDR/NoC 먼저 alive → Core / PE가 나중에 reset 해제
  - 안 그러면 core가 AXI 트랜잭션 쏘는데 NoC 쪽은 아직 reset 상태 이런 상황 발생

### 3-2. CDC FIFO reset 규칙

- dual-clock FIFO 구조:

```
wr_clk domain: reset_sync(wr_clk)
rd_clk domain: reset_sync(rd_clk)
```

- 둘 다 각자 domain reset을 sync해야 함
- 그리고 `empty`, `full` flag는
   **reset 후 일정 cycle 동안 강제로 safe한 상태** 만드도록 구현 (vendor FIFO IP는 보통 알아서 해줌)

------

## 4. FPGA Prototype에서 Async → Sync-only Reset 전환

ASIC 코드 그대로 FPGA에 올리면:

- async reset + huge fanout → routing/hold 문제, Fmax 저하
- FPGA마다 **Global Set/Reset(GSR)**가 있어서 power-up 시 기본 초기화 지원하는 경우 많음

### 4-1. 공통 RTL에서 ASIC/FPGA 분기하는 방법

```
// 정의부 (package나 global 헤더)
`define ASIC 1
// `define FPGA 1

// 예시 reset block
module my_block (
    input  logic clk,
    input  logic rst_n_in,  // top에서 준 reset (ASIC: async, FPGA: sync)
    ...
);

    logic rst_n;

`ifdef ASIC
    // ASIC: async assert / sync deassert 사용
    reset_sync u_rst_sync (
        .clk         (clk),
        .rst_n_async (rst_n_in),
        .rst_n_sync  (rst_n)
    );
`else
    // FPGA: 이미 rst_n_in이 sync라고 가정하고 바로 사용
    assign rst_n = rst_n_in;
`endif

    always_ff @(posedge clk) begin
        if (!rst_n) begin
            // reset 동작
        end else begin
            // normal 동작
        end
    end

endmodule
```

FPGA용 top에서는:

- tool에서 global reset network를 설정
- 가능하면 **sync reset**으로 넣고,
   GSR + initial value로 파이프라인 data reg는 굳이 reset 안 걸기.

------

## 5. Synopsys DC / PrimeTime / Innovus – Reset 관련 Best Practice

### 5-1. Design Compiler (DC) – Reset 네트워크 취급

보통 다음처럼 함:

1. reset sync까지의 경로는 timing 무시
2. 그 이후(즉, `rst_n_sync`부터)는 **일반 data로 STA**

예:

```
# reset은 data path가 아니므로
set_false_path -from [get_ports por_n] -to [all_registers]

# 또는 reset_sync 내부의 async pin에 대해 타이밍 끄기
set_disable_timing [get_pins reset_sync*/rst_n_async]
```

- 가능하면 `reset_sync` 모듈 이름 통일해서 script에서 잡기 좋게.

### 5-2. PrimeTime – Recovery/Removal 처리

- 우리가 **sync release** 구조를 쓰면,
  - 실제 async reset은 sync flop까지 딱 한 단계
  - 그 이후는 synchronous reset으로만 propagation → recovery/removal 신경 덜 씀
- 여전히 필요하면:

```
# Recover/Removal Violation을 보고 싶으면 기본 세팅 유지
# 보기 싫으면 (권장X)
set_false_path -from [get_ports por_n]
```

> 진짜 권장 방식은 “메시지를 없애는 것”이 아니라
>  **reset_sync 구조를 도입해서 문제를 원천 제거**하는 것.

### 5-3. Innovus / PnR – Reset Routing

- reset을 **clock처럼 취급하진 않지만, 여전히 high-fanout net**

- 흔한 세팅:

  ```
  set_max_fanout 32 [get_nets rst_n_*]
  ```

- 필요하면 **특별한 routing layer**나 buffer tree를 지정해서 skew를 줄임

- 하지만 우리는 domain별로 reset_sync를 넣었기 때문에,
   global reset skew가 조금 있어도 문제 적음 (release는 이미 sync)

------

## 6. Questa vs VCS – Recovery/Removal 메시지

### 6-1. 현상

- async reset을 직접 FF에 연결하고
   reset deassert를 clock과 애매하게 하면:
  - VCS: `UVM_WARNING`, `$recovery` / `$removal` violation 비슷한 메시지
  - Questa: `(vsim-3049) $recovery` / `(vsim-3048) $removal` 등등

### 6-2. 타이밍 체크 끄는 옵션 (알고만 있고, 남용 X)

- Questa:
  - `+notimingchecks` : 모든 타이밍 체크 off (비추천)
  - 또는 `specify` 블록에서 개별 셀에 대해 `$recovery`, `$removal` 삭제
- VCS:
  - `+no_tchk_msg` / `+neg_tchk` 등의 옵션 존재

**하지만 진짜 해결책은**:

- async reset을 **오직 synchronizer FF들에만 연결**
- 나머지는 `rst_n_sync`만 사용
   → Recovery/Removal violation 자체를 설계 구조로 없애버림.

------

## 7. 최종 요약 – 네 프로젝트 기준 Reset 전략

너 상황(ASIC ISP + systolic array + FPGA proto + 다양한 tool)에서 쓸 수 있는 **실질 가이드**만 압축해보면:

1. **SoC Top**
   - `por_n` → 각 clock domain마다 `reset_sync` → `*_rst_n_sync`만 사용
2. **ISP / Imaging pipeline**
   - Data path는 가능하면 reset 안 하고 `valid` 플래그만 reset
   - FSM, config reg는 반드시 sync reset
3. **Multi-PE Systolic / CDC**
   - 각 domain마다 `reset_sync`
   - reset release 순서 정의 (NoC/DDR → Core/PE)
   - dual-clock FIFO는 양쪽 domain reset 모두 sync
4. **FPGA Prototype**
   - 가능하면 sync-only reset 사용
   - 공통 RTL에서 `ASIC/FPGA` macro로 분기
5. **Tool (DC/PT/Innovus)**
   - async reset port → `set_false_path` or `set_disable_timing`
   - reset_sync 이후는 일반 data로 STA
6. **Simulator (Questa/VCS)**
   - 타이밍 옵션으로 가리는 것보다
   - reset 구조(2FF sync)로 recovery/removal 문제 자체를 제거



# Part 4. 추가



## 1. Multi-domain Bring-up Sequence 예제

*(cfg_done handshake 이후에 core reset release)*

### 1-1. 시나리오 설정

- 도메인 1: **cfg_clk domain**
  - 레지스터 설정 / 초기화만 담당
  - 외부에서 APB, SPI, I2C 등으로 설정값을 받아 내부 레지스터 구성
  - 설정 끝나면 `cfg_done` = 1
- 도메인 2: **core_clk domain**
  - 실제 데이터 path / PE / ISP core 동작
  - **설정이 끝나기 전에는 reset 상태로 묶어둬야 안전**
- 공통 reset:
  - 외부 `por_n` (power-on reset, async)
  - 각 domain 별로 `reset_sync`를 통해 **async assert / sync deassert**

### 1-2. 전체 bring-up 시퀀스 (개념)

1. Power up 시 `por_n = 0`
    → cfg/core 모든 FF reset
2. `por_n`이 1로 올라감
   - cfg_rst_n_sync, core_rst_n_sync_raw 각각 자기 클록에 대해 sync release
   - **하지만 core side는 “raw sync reset” 이고, 실제 core_rst_n은 아직 잡아둘 수 있음**
3. cfg domain 활성화
   - cfg FSM이 동작해서 각종 설정 레지스터 write
   - 설정 완료 시점에 `cfg_done`을 1로 세우고 **고정(high) 유지**
4. `cfg_done`을 core domain에서 2FF sync
   - `cfg_done_sync` 생성
5. core domain reset controller가:
   - `core_rst_n_sync_raw == 1`
   - `cfg_done_sync == 1`
      두 조건을 만족하면 **몇 cycle 딜레이 후 core_rst_n을 1로 release**
6. core block이 설정된 config를 가지고 정상 동작 시작

### 1-3. 블록/코드 예제

#### (1) Top 레벨 구조

```
module top (
    input  logic por_n,
    input  logic cfg_clk,
    input  logic core_clk
);

    // 1) 도메인별 reset sync
    logic cfg_rst_n_sync;
    logic core_rst_n_sync_raw;

    reset_sync u_cfg_rst_sync (
        .clk         (cfg_clk),
        .rst_n_async (por_n),
        .rst_n_sync  (cfg_rst_n_sync)
    );

    reset_sync u_core_rst_sync (
        .clk         (core_clk),
        .rst_n_async (por_n),
        .rst_n_sync  (core_rst_n_sync_raw)
    );

    // 2) cfg domain: 설정 + cfg_done 생성
    logic cfg_done; // cfg_clk domain

    cfg_block u_cfg (
        .clk      (cfg_clk),
        .rst_n    (cfg_rst_n_sync),
        .cfg_done (cfg_done)
    );

    // 3) cfg_done을 core_clk domain으로 동기화
    logic cfg_done_sync;

    sync_2ff u_cfg_done_sync (
        .clk      (core_clk),
        .din      (cfg_done),
        .dout     (cfg_done_sync)
    );

    // 4) core reset controller (core_clk domain)
    logic core_rst_n; // 최종 core reset (sync)

    core_reset_ctrl u_core_rst_ctrl (
        .clk               (core_clk),
        .rst_n_sync_raw    (core_rst_n_sync_raw),
        .cfg_done_sync     (cfg_done_sync),
        .core_rst_n        (core_rst_n)
    );

    // 5) core block
    core_block u_core (
        .clk   (core_clk),
        .rst_n (core_rst_n)
    );

endmodule
```

`sync_2ff`는 일반적인 2단 동기화 모듈:

```
module sync_2ff (
    input  logic clk,
    input  logic din,
    output logic dout
);
    logic [1:0] ff;

    always_ff @(posedge clk) begin
        ff <= {ff[0], din};
    end

    assign dout = ff[1];

endmodule
```

#### (2) cfg_block (설정 + cfg_done)

```
module cfg_block (
    input  logic clk,
    input  logic rst_n,
    output logic cfg_done
);

    typedef enum logic [1:0] {
        IDLE,
        LOAD_CFG,
        DONE
    } state_t;

    state_t state, next;

    // 상태 전이
    always_ff @(posedge clk) begin
        if (!rst_n)
            state <= IDLE;
        else
            state <= next;
    end

    always_comb begin
        next = state;
        case (state)
            IDLE: begin
                // 외부 cfg start 조건이 있으면 사용
                next = LOAD_CFG;
            end
            LOAD_CFG: begin
                // 레지스터 write 등 완료되면 DONE으로
                // (여기선 예제로 바로 DONE)
                next = DONE;
            end
            DONE: begin
                next = DONE;
            end
        endcase
    end

    // cfg_done 생성
    always_ff @(posedge clk) begin
        if (!rst_n)
            cfg_done <= 1'b0;
        else if (state == DONE)
            cfg_done <= 1'b1; // 이후 계속 1 유지
    end

endmodule
```

#### (3) core_reset_ctrl (cfg_done 이후 reset release)

```
module core_reset_ctrl (
    input  logic clk,
    input  logic rst_n_sync_raw, // reset_sync에서 나온 "기본" sync reset
    input  logic cfg_done_sync,  // cfg_done의 core_clk 동기 버전
    output logic core_rst_n      // 최종 core reset
);

    // 간단 예: cfg_done_sync == 1을 본 뒤 몇 cycle 후 reset 해제
    typedef enum logic [1:0] {
        RST_HOLD,
        WAIT_CFG_DONE,
        RELEASE_DELAY,
        RUN
    } state_t;

    state_t state, next;
    logic [3:0] delay_cnt;

    // state FF
    always_ff @(posedge clk) begin
        if (!rst_n_sync_raw) begin
            state     <= RST_HOLD;
            delay_cnt <= '0;
        end else begin
            state     <= next;
            if (state == RELEASE_DELAY)
                delay_cnt <= delay_cnt + 1'b1;
            else
                delay_cnt <= '0;
        end
    end

    // next state
    always_comb begin
        next = state;
        case (state)
            RST_HOLD: begin
                // raw reset은 풀렸지만, 아직 core는 hold
                if (rst_n_sync_raw)
                    next = WAIT_CFG_DONE;
            end
            WAIT_CFG_DONE: begin
                if (cfg_done_sync)
                    next = RELEASE_DELAY;
            end
            RELEASE_DELAY: begin
                if (delay_cnt == 4'd3) // 4 cycle 정도 기다림 (예시)
                    next = RUN;
            end
            RUN: begin
                next = RUN;
            end
        endcase
    end

    // 최종 reset 생성
    always_ff @(posedge clk) begin
        if (!rst_n_sync_raw)
            core_rst_n <= 1'b0;
        else begin
            case (next)
                RST_HOLD,
                WAIT_CFG_DONE,
                RELEASE_DELAY: core_rst_n <= 1'b0;
                RUN:           core_rst_n <= 1'b1;
            endcase
        end
    end

endmodule
```

> 이 패턴이 **multi-domain bring-up에서 가장 많이 쓰는 형태**:
>
> - 각 도메인 reset은 sync 처리
> - 기능 domain은 “설정 domain이 cfg_done 찍을 때까지” reset hold
> - 필요하면 “역방향 handshake(core_ready)”도 정의해서 cfg 쪽이 마지막 check하고 normal run 진입.



# Part 5. 면접 관련

## 2. Reset 관련 면접 질문 리스트 + 모범 답변

아래는 **디지털/ASIC/SoC 인터뷰에서 실제로 나올 법한 reset 질문들**을
 당신 스타일에 맞게 **짧고 기술적으로 정확하게** 답변한 버전이에요.

------

### Q1. Async reset과 Sync reset의 차이 및 장단점은?

**A.**

- **Async reset**은 클록과 무관하게 FF를 즉시 초기화하고, assert 시점이 빠르고 확실하다는 장점이 있습니다. 하지만 **release가 clock edge 근처에서 일어나면 metastability와 recovery/removal 문제**가 생깁니다.
- **Sync reset**은 클록 엣지에서만 동작해서 reset 트랜지션이 항상 clock에 정렬되므로 **타이밍 분석이 단순**하고 glitch에 강합니다. 다만 clock이 살아 있어야 reset이 풀리기 때문에, power-up 초기에 clock이 없는 환경에서는 단독으로 쓰기 어렵습니다.

## **Q1. What is the difference between asynchronous reset and synchronous reset?**

**A:**
 An asynchronous reset is asserted and deasserted independently of the clock. It’s useful during power-up because the clock may not be stable yet.
 A synchronous reset only changes state on the active clock edge, which makes timing analysis simpler and ensures that the reset transition is aligned with the clock.

Async reset is great for fast, reliable assertion, but the deassertion is dangerous because it can violate recovery/removal timing. Sync reset avoids that issue, but it requires a running clock.

------

### Q2. 왜 “async assert, sync deassert” 패턴을 쓰나요?

**A.**
 Power-up 직후에는 clock이 안정적이지 않거나 없을 수 있어서 **fast/robust한 async assert가 필요**합니다. 하지만 reset을 비동기로 풀면 각 flip-flop이 서로 다른 시점에 reset이 풀려 FSM, 파이프라인이 **불일치 상태**로 시작할 수 있고, metastability도 발생합니다. 그래서 보통 **reset 입력은 async로 받고, 내부에서는 2FF synchronizer를 통해 sync deassert**하여 두 장점만 취하는 패턴을 씁니다.

## **Q2. Why do we use the “asynchronous assert, synchronous deassert” style?**

**A:**
 We need the asynchronous assertion for power-up initialization, but releasing reset asynchronously is unsafe.
 If reset is deasserted close to a clock edge, the flip-flop can enter metastability or different flops may come out of reset in different cycles.
 So the common industry pattern is:

- Accept async reset at the input
- Pass it through a 2-FF synchronizer
- Use the synchronized version for all internal logic

This gives the benefit of fast assertion while guaranteeing a clean, clock-aligned release.

------

### Q3. Metastability가 무엇이며, reset과 어떤 관련이 있나요?

**A.**
 Metastability는 flip-flop 입력이 setup/hold window 근처에서 바뀔 때 출력이 0/1 사이의 불안정 상태에 머무는 현상입니다. reset deassert가 clock edge와 비슷한 타이밍으로 일어나면 reset input도 사실상 “데이터”처럼 보이기 때문에 **reset을 본 FF가 metastable**해질 수 있습니다. 그래서 reset release도 clock에 맞춰서 synchronizer를 통해 처리합니다.

## **Q3. How does metastability relate to reset deassertion?**

**A:**
 If reset deasserts very close to the clock edge, it violates the FF’s recovery or removal timing. From the FF’s perspective, reset is just another asynchronous control signal, so it can place the flop into a metastable state—neither ‘0’ nor ‘1’.

That metastability can propagate into state machines or pipeline control signals and cause unpredictable behavior.
 Using a synchronizer for reset deassertion prevents this.

------

### Q4. Recovery/Removal 타이밍은 무엇인가요?

**A.**

- **Recovery time**: async control 신호(예: reset)가 deassert된 후, 다음 clock edge까지 필요한 최소 시간
- **Removal time**: clock edge 이후, async control 신호가 deassert되기까지 유지해야 하는 최소 시간
   둘 중 하나라도 위반되면 FF가 metastability에 빠질 수 있습니다. reset을 async로 release하면 이 조건을 지키기 어렵기 때문에, sync release를 통해 문제를 줄입니다.

## **Q4. What are recovery and removal times?**

**A:**
 They are timing requirements specific to asynchronous control signals:

- **Recovery time** is the minimum time between reset deassertion and the next clock edge.
- **Removal time** is the minimum time after a clock edge before reset can be deasserted.

Async reset release must satisfy both, otherwise the FF can become metastable.
 Synchronizing the reset deassertion makes these checks trivial.

------

### Q5. Multi-clock SoC에서 reset을 어떻게 설계하나요?

**A.**
 각 clock domain마다 **별도의 reset synchronizer**를 두고, 외부 reset은 **각 domain의 local sync reset으로 변환**해서 사용합니다. Domain 간 의존성이 있을 경우, 하나의 도메인(예: NoC/DDR)이 정상 동작 상태가 된 이후 다른 도메인(예: Core/PE) reset을 풀도록 **bring-up sequence를 설계**합니다. Cross-domain status 신호는 모두 2FF sync를 거쳐 사용합니다.

## **Q5. How do you handle reset distribution in a multi-clock SoC?**

**A:**
 Each clock domain must have its own reset synchronizer.
 The top-level reset (POR or external reset) fans out to each domain, and then each domain locally creates a clean, synchronized reset.

If there are dependencies between domains, we define an explicit **bring-up sequence**. For example, the NoC or DDR domain might come out of reset first, and the compute or ISP domain is released only after receiving a synchronized `cfg_done` or `ready` signal from another domain.

------

### Q6. Reset synchronizer는 왜 보통 2단인가요? 3단이면 안 되나요?

**A.**
 2FF synchronizer는 첫 번째 FF에서 metastability가 발생해도, 두 번째 FF에는 metastability가 전이될 확률을 크게 줄여주며, 대부분의 시스템 MTBF 요구 사항을 만족합니다. MTBF 요구가 매우 엄격한 시스템(고속/mission-critical)에서는 3단을 쓰기도 합니다. 결국 trade-off는 **latency 1 cycle vs. MTBF**입니다.

## **Q6. Why do we generally use 2-FF reset synchronizers?**

**A:**
 The first FF may go metastable when reset is deasserted.
 The second FF provides an additional cycle for the metastability to resolve, significantly improving MTBF.

Two stages meet the MTBF needs of most designs, but in extremely high-reliability systems we may use three.
 It’s a latency vs. MTBF trade-off.

------

### Q7. 모든 flip-flop에 reset을 걸어야 하나요?

**A.**
 아닙니다. 보통 **control/FSM/상태 레지스터**는 deterministic한 초기 상태가 필요하기 때문에 reset이 꼭 필요합니다. 반면 data path pipeline 레지스터는 reset 대신 **valid 플래그를 reset**하고, reset 이후 일정 기간 데이터를 무시하는 방식으로 설계하면 area와 routing을 줄일 수 있습니다. ASIC/SoC에서는 이런 selective reset 전략이 매우 중요합니다.

## **Q7. Should every flip-flop in the design have a reset?**

**A:**
 Not necessarily.
 Only control, state machines, and critical configuration registers must have deterministic reset values.

Data-path pipelines often do **not** need reset because:

- They can be flushed using a `valid` signal
- Resetting thousands of data flops increases area, power, and routing congestion

This selective-reset strategy is very common in ASICs.

------

### Q8. Clock gating이나 저전력 블록에서 reset은 어떻게 처리하나요?

**A.**
 Clock gated domain은 clock이 꺼져 있으면 sync reset이 전달되지 않으므로, reset이 동작해야 할 때는 **gating을 강제로 open**하여 clock을 흘려보내거나, 상위 ungated clock domain에서 reset sequence를 제어해야 합니다. 일반적인 패턴은 `g_en_safe = g_en | ~rst_n_sync;`처럼 reset 동안에는 gating을 무시하고 clock을 켜두는 방식입니다. Reset 후 정상 동작에 들어가면 다시 원래 gating 조건으로 돌아갑니다.

## **Q8. How do you handle reset in clock-gated or low-power domains?**

**A:**
 A synchronous reset requires a running clock to propagate, so during reset, clock gating must be temporarily bypassed.
 A typical pattern is:

```
g_en_safe = g_en | ~rst_n_sync;
```

This ensures that gated clocks are forced ON while reset is active. After the reset is released, normal clock gating takes over.

------

### Q9. FPGA와 ASIC에서 reset 전략의 차이는 무엇인가요?

**A.**
 FPGA는 내부 FF에 대한 **power-up initial value**를 지원하고, Global Set/Reset(GSR) 같은 구조가 있기 때문에, 많은 경우 굳이 async reset을 사용하지 않고 **sync reset 또는 initial value 기반**으로 설계하는 것이 흔합니다. ASIC은 power-up 시 초기값이 정의되지 않기 때문에, POR + async reset이 필요하고, 이후 sync deassert를 통해 타이밍·안정성을 확보합니다. 또한 ASIC에서는 reset routing이 매우 큰 fanout을 가지므로 reset tree 구조도 중요한 설계 요소입니다.

## **Q9. What is the difference between ASIC and FPGA reset strategies?**

**A:**
 ASICs require explicit resets because flip-flop power-up states are undefined.
 FPGA flip-flops typically have defined power-up values and global GSR networks, so synchronous resets or even “no reset + initial value” designs are common.

FPGA resets must consider routing delays and fan-out; thus synchronous resets can help meet timing, whereas ASIC resets must consider recovery/removal timing and large reset trees.

------

### Q10. Reset domain crossing(RDC)은 뭐고, 어떻게 다루나요?

**A.**
 RDC는 서로 다른 reset 신호(또는 release 순서)를 가진 블록들 사이의 신호 전달을 말합니다. 하나의 domain은 reset 상태인데 다른 domain은 정상 동작이면, handshake나 상태 신호가 이상한 순서로 전달될 수 있습니다. 해결책은 **reset release 순서를 의도적으로 설계**하고, cross-domain 신호는 항상 synchronizer와 safe handshake 프로토콜(예: valid/ready, req/ack)로 감싸는 것입니다.

## **Q10. What is Reset Domain Crossing (RDC)?**

**A:**
 RDC occurs when two logic blocks use different reset signals or release reset at different times. This can cause handshake signals to be sampled while one side is still in reset.

To avoid RDC issues, we:

- Synchronize all cross-domain status signals
- Define a safe reset release sequence
- Ensure both domains only exchange traffic after both are operational

------

### Q11. CDC FIFO나 dual-clock FIFO의 reset은 어떻게 해야 하나요?

**A.**
 Write clock domain과 read clock domain 각각에서 **독립적인 reset synchronizer**를 사용해 write-side 및 read-side pointer를 초기화합니다. FIFO 내부의 flag(empty/full)는 reset 시 항상 safe한 상태가 되도록 설계해야 하며, 보통 vendor IP는 이 부분을 잘 처리해 줍니다. 중요한 것은 reset sequence 중에 FIFO 입출력 신호가 이상한 패턴으로 튀지 않도록 **reset 동안에는 write/read를 금지**하는 상위 로직입니다.

## **Q11. How do you reset dual-clock FIFOs or CDC FIFOs?**

**A:**
 Each side of the FIFO—write and read—must be reset and synchronized **independently** to its own clock.
 The write pointer and read pointer are reset in their respective domains, and the flags handle the safe relationship between them.

During reset, both write and read operations must be inhibited to avoid corrupted pointer transitions.

------

### Q12. UVM/Testbench에서 reset을 어떻게 다루나요?

**A.**
 UVM에서는 보통 `uvm_reset_phase` 또는 custom reset sequence를 사용해서 **reset assert/deassert, configuration, start-of-sim**을 관리합니다. Reset 동안에는 driver가 transaction을 보내지 않도록 하고, reset 이후에 sequencer를 start합니다. 또한 DUT와 testbench의 reset 타이밍을 맞추기 위해 virtual interface의 reset 신호를 TB에서 직접 제어하거나, top-level clock/reset generator를 사용합니다.

## **Q12. How do you handle reset in UVM testbench environments?**

**A:**
 Typically we drive reset from a top-level clock/reset generator and connect it to the DUT via virtual interfaces.
 During `uvm_reset_phase`, we assert reset, wait several cycles, deassert it synchronously, and then start sequences.

Drivers must avoid sending transactions during reset, and monitors typically ignore traffic until reset is cleared.

------

### Q13. Reset 관련해서 자주 발생하는 bug 예시는?

**A.**

- Reset release 타이밍이 도메인마다 달라 FSM이 불일치 상태에서 시작하는 문제
- CDC 경로에서 한쪽 domain은 reset 상태, 다른 한쪽은 active 상태라 handshake가 꼬이는 문제
- Async reset을 SVA/STA에서 제대로 처리하지 않아 recovery/removal violation이 뒤늦게 터지는 경우
- FPGA에서 reset fanout이 너무 크고 routing이 꼬여 Fmax가 떨어지는 문제
   이런 이슈를 줄이기 위해 **async assert, sync deassert + domain별 reset sync + bring-up sequence 정의**를 일관되게 적용하는 것이 중요합니다.

## **Q13. What are common bugs caused by improper reset handling?**

**A:**

- FSMs coming out of reset in different cycles → inconsistent starting state
- One domain alive while another is still in reset → handshake deadlocks
- Recovery/removal violations during async release → metastability
- FPGA designs failing timing because reset fanout is too large

Most of these problems disappear when using a consistent scheme:
 **async assert → sync deassert → domain-local synchronizers → explicit bring-up sequencing.**





# 기타

https://www.youtube.com/watch?v=3A5Yy-UXi-4

`TODO`: Metastability일때, 실제 중간 전압에 머무를 수 있나?

그래서 0도 1도 아닌 상태가 될 수 있는건가?

아니면, 실제로는 0 아니면 1로 되지만, 그 값이 random이라는 말인가?





# 참고

참고

http://www.sunburst-design.com/papers/CummingsSNUG2003Boston_Resets.pdf

