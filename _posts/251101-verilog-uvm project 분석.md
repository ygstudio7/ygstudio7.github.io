# 📁 권장 디렉터리 구조

```
project_root/
├─ rtl/
│  ├─ if_stream.sv             : interface 정의
│  ├─ dut_invert.sv            : 테스트할 device
└─ uvm/                      
   ├─ uvm_env_pkg.sv         :   <-- 패키지 본체 (여기서 .svh 들을 include)
   ├─ pixel_tr.svh            : 한 픽셀을 나타내는 transaction 객체
   ├─ pixel_sequencer.svh    : sequence -> driver 연결
   ├─ pixel_driver.svh        : tb 입력 구동기, in_valid, in_pix 등 신호 생성
   ├─ out_monitor.svh        : DUT의 출력 (out_pix)를 관찰, transaction 생성후 scoreboard로 전달
   ├─ img_scoreboard.svh    : 골든 벡터 (frame_out_golden.raw) 와 DUT 출력 비교
   ├─ input_agent.svh        : sequence + driver 묶음
   ├─ env.svh                : 전체 환경 구성 (agent, monitor, scoreboard 연결)
   ├─ feed_raw_seq.svh        : 입력 영상 (frame_in.raw) 를 순차적으로 보내는 시퀀스
   └─ test_basic.svh        : 메인 test 클래스 (env 인스턴스 생성 및 실행 제어)
├─ scripts/
│  └─ generate_vectors.py
├─ vec/
│  ├─ frame_in.raw
│  └─ frame_out_golden.raw
├─ run.bat
├─ run_gui.bat
└─ run_win.do
```

[FIXME] 위 파일들 간단 설명을 디렉터리에 추가

# 0) 한눈 요약 (데이터 흐름)

sddms 구조

`frame_in.raw  ──▶ [Sequence] ─▶ [Driver] ─▶ DUT(invert) ─▶ [Monitor] ─▶ [Scoreboard]                                                     │ frame_out_golden.raw  ───(시작 시 로드, Queue) ─────┘       (pop_front 비교)`

- 입력 프레임을 바이트 스트림으로 DUT에 흘려주고, DUT 출력이 나올 때마다 **골든 큐의 맨 앞(pop_front)** 과 즉시 비교합니다.

- 파이프라인 지연은 “출력 valid 시점 기준 비교”로 자연 정렬됩니다.

- 

## 🔶 1. `if_stream.sv` — 인터페이스 정의

### 🎯 목적

DUT(디자인)과 테스트벤치 간에 신호를 교환하기 위한 **표준 스트리밍 인터페이스**를 정의합니다.

### 📘 주요 내용

| 항목                                  | 설명                                  |
| ----------------------------------- | ----------------------------------- |
| `in_valid`, `in_ready`, `in_pix`    | 테스트벤치가 DUT에 입력하는 신호 (Input Stream)  |
| `out_valid`, `out_ready`, `out_pix` | DUT가 테스트벤치에 출력하는 신호 (Output Stream) |
| `modport drv`                       | 드라이버가 사용하는 방향 정의 (TB → DUT)         |
| `modport mon_o`                     | 모니터가 사용하는 방향 정의 (DUT → TB)          |

### 💡 포인트

- **하나의 인터페이스 객체(`sif`)**가 DUT과 TB를 연결함.
- `in_ready/out_ready`를 interface 내부에서 assign 하지 않고,
   DUT나 TB 쪽에서 구동해야 **다중 드라이브 충돌이 안 생김.**

### 설명

(1) 인터페이스 이름: if_stream, 

파라미터 PIXW: 픽셀 데이터 폭, 

인자: clk, rst_n

(2) 입력 스트림

(3) 출력 스트림

(4) driver용 방향 설정: 출력쪽이므로, in_valid, in_pix 를 내보내고, in_ready를 받음

(5) monitor는 모두가 입력

```
interface if_stream #(parameter int PIXW=8) (input logic clk, input logic rst_n); //(1)
  // Input stream (TB -> DUT)        // (2)
  logic              in_valid,  in_ready;
  logic [PIXW-1:0]   in_pix;

  // Output stream (DUT -> TB)     // (3)
  logic              out_valid, out_ready;
  logic [PIXW-1:0]   out_pix;

  // UVM driver side (produces input stream)       // (4)
  modport drv   (input  clk, rst_n, in_ready,
                 output in_valid, in_pix);

  // UVM monitor side (observes DUT output)        // (5)
  modport mon_o (input clk, rst_n, out_valid, out_pix, out_ready);
endinterface


```



## 🔶 2. `dut_invert.sv` — Device Under Test (테스트 대상)

### 🎯 목적

영상의 각 픽셀 값을 반전(invert)하는 RTL 예제.
 (예: `out_pix = 255 - in_pix`)

### 📘 주요 내용

| 항목                    | 설명                      |
| --------------------- | ----------------------- |
| `PIXW`                | 픽셀 데이터 폭(비트 수), 기본 8bit |
| `in_valid/in_ready`   | 입력 유효/준비 신호             |
| `out_valid/out_ready` | 출력 유효/준비 신호             |
| 내부 레지스터 `v_q`, `d_q`  | 파이프라인용 유효신호/데이터 저장      |

### 💡 포인트

- `in_ready=1’b1`로 고정 → 항상 입력 가능
- 한 사이클 파이프라인 구조
- 실제 영상처리 DUT을 만들 때는 여기서 convolution, filter, etc.가 들어감



### 코드

(1) 입력 픽셀폭을 PIXW 파라미터로 정의

(2) 스트리밍 입출력 정의

(3)  출력은 1 clock 지연

(4) 출력을 입력의 반전값으로 내보냄

```
module dut_invert #(
  parameter int PIXW = 8                 // (1)
)(
  input  logic              clk,        // (2)
  input  logic              rst_n,
  input  logic              in_valid,
  output logic              in_ready,
  input  logic [PIXW-1:0]   in_pix,
  output logic              out_valid,
  input  logic              out_ready,
  output logic [PIXW-1:0]   out_pix
);

  // FIXME:
  assign in_ready = 1'b1;

  // 1-stage pipeline (valid/data register)
  logic              v_q;
  logic [PIXW-1:0]   d_q;

  always_ff @(posedge clk or negedge rst_n) begin
    if (!rst_n) begin
      v_q <= 1'b0;
      d_q <= '0;
    end else begin
      // 1 clock delay                    // (3)
      v_q <= in_valid;
      d_q <= in_pix;
    end
  end

  // invert
  assign out_valid = v_q;
  assign out_pix   = {PIXW{1'b1}} - d_q;  // e.g., 8b: 255 - d_q    // (4)

endmodule


```



## 🔶 3. `tb_top.sv` — 최상위 테스트벤치

### 🎯 목적

- 클록/리셋 생성
- 인터페이스 및 DUT 인스턴스 연결
- UVM 설정 (`config_db`) 등록
- 테스트 실행 (`run_test("test_basic")`)

### 📘 주요 동작

| 구분                       | 설명                           |
| ------------------------ | ---------------------------- |
| `clk`, `rst_n`           | TB용 기본 클럭 및 리셋 생성            |
| `if_stream sif(.*)`      | 인터페이스 생성, 자동 연결              |
| `sif.out_ready` 랜덤 토글    | DUT 백프레셔(backpressure) 발생    |
| `dut_invert` 인스턴스        | 실제 DUT 연결                    |
| `uvm_config_db::set()`   | virtual interface 및 파일 경로 등록 |
| `run_test("test_basic")` | 지정한 테스트 시작                   |



### 코드

(1) uvm을 쓰겠다는 선언문이다. uvm_macros.svh와 uvm_pkg는 늘 함께 다닌다. uvm 프레임워크를 자체 패키지로, systemverilog로 작성된 uvm 핵심 클래스들이 들어 있다. 

qustasim의 경우, $UVM_HOME/src/uvm_pkg.sv 로 설정되고, (`UVM_HOME`은 예: `C:/intelFPGA_pro/24.2/questa_fe/verilog_src/uvm-1.2`) 로 설정해서 사용할 수 있다.

여기에는 보통 다음과 같은 대표 클래스들이 들어있다.

| 클래스                                                             | 설명                        |
| --------------------------------------------------------------- | ------------------------- |
| `uvm_component`                                                 | 모든 UVM 컴포넌트의 부모 클래스       |
| `uvm_sequence_item`                                             | 드라이버로 전달되는 데이터 객체의 기본형    |
| `uvm_sequence`                                                  | 트랜잭션을 순차적으로 발생시키는 베이스 클래스 |
| `uvm_driver`, `uvm_monitor`, `uvm_agent`, `uvm_env`, `uvm_test` | 검증 환경 구성 요소들의 베이스 클래스     |
| `uvm_config_db`                                                 | TB 설정값 전달용 데이터베이스         |
| `uvm_analysis_port`, `uvm_subscriber`                           | 모니터 → 스코어보드 데이터 전달용 인터페이스 |
| `uvm_factory`, `uvm_object_utils`, `uvm_component_utils`        | 클래스 등록 및 생성 시스템           |

(2) uvm_env_pkg는 **사용자 정의 테스트 환경**들로, 이 프로젝트만의 클래스들을 모아놓은 것

```
UVM Core (from uvm_pkg)
│
├── uvm_object
│   ├── uvm_sequence_item         : 실제 전송될 데이터의 기본            
│   │   └── pixel_tr              : 하나의 픽셀값을 표현하는 사용자 정의 트랜잭션
│   ├── uvm_sequence              : 드라이버에 순차적으로 트랜잭션을 생성해서 보내는 시퀀스
│   │   └── feed_raw_seq          : 입력 raw 파일을 읽어 트랜잭션으로 변환
│   └── uvm_subscriber            : 모니터에서 보낸 데이터 스트림을 받아 처리하는 클래
│       └── img_scoreboard        : 수신된 픽셀을 골든 데이터와 비교
│
├── uvm_component
│   ├── uvm_sequencer             : 시퀀스와 드라이버 사이의 인터페이스            
│   │   └── pixel_sequencer         : 픽셀 트랜잭션을 순서대로 드라이버에 전달
│   ├── uvm_driver                
│   │   └── pixel_driver            ← (픽셀 신호 구동)
│   ├── uvm_monitor
│   │   └── out_monitor             ← (출력 모니터)
│   ├── uvm_agent
│   │   └── input_agent             ← (시퀀서 + 드라이버 묶음)
│   ├── uvm_env
│   │   └── env                     ← (agent + monitor + scoreboard 통합 환경)
│   └── uvm_test
│       └── test_basic              ← (메인 테스트)
│
└── uvm_root (자동 생성)
    └── uvm_test_top (run_test로 실행되는 최상위 UVM 노드)
```

(2) 클럭 및 리셋 생성

(3) 인터페이스 인스턴스 생성하고,  `.*` 으로 clk, rst_n 자동 연결 (이름이 같으면 연결되는 듯)

(4) out_ready를 20% 비율로 0으로 만들어, DUT에 출력 지연 상황 발생 (**back pressure 발생**)

(5) dut 생성하고, 인터페이스 연결

(6) 여기서 하는 일은 3가지

* virtual interface를 UVM 환경인  env에 넘기고

* 입력/골든 파일 경로를 test에 넘기고

* test_basic이라는 UVM test 클래스를 실행한다.

```
`timescale 1ns/1ps
`include "uvm_macros.svh"                                 // (1)
import uvm_pkg::*;
import uvm_env_pkg::*;

module tb_top;

  // -------------------- Clock / Reset -------------------- // (2)
  logic clk;
  logic rst_n;

  initial begin
    clk = 0;
    forever #5 clk = ~clk;  // 100 MHz
  end

  initial begin
    rst_n = 0;
    #100;
    rst_n = 1;
  end

  // -------------------- Interface --------------------
  if_stream #(8) sif (.*);   // clk, rst_n: Auto connection      // (3)

  // -------------------- Random out_ready Toggle -------------------- // (4)
  initial begin
    sif.out_ready = 1'b1;
    wait (rst_n === 1);
    forever begin
      @(posedge clk);
      // 20% 확률로 backpressure
      if ($urandom_range(0, 9) < 2)
        sif.out_ready <= 1'b0;
      else
        sif.out_ready <= 1'b1;
    end
  end

  // -------------------- DUT -------------------- // (5)
  dut_invert #(.PIXW(8)) dut_i (
    .clk        (clk),
    .rst_n      (rst_n),
    .in_valid   (sif.in_valid),
    .in_ready   (sif.in_ready),
    .in_pix     (sif.in_pix),
    .out_valid  (sif.out_valid),
    .out_ready  (sif.out_ready),
    .out_pix    (sif.out_pix)
  );

  // -------------------- UVM Config -------------------- // (6)
  initial begin
    // virtual interface & file path
    uvm_config_db#(virtual if_stream#(8))::set(null, "uvm_test_top.m_env", "vif", sif);
    uvm_config_db#(string)::set(null, "uvm_test_top", "in_file",     "vec/frame_in.raw");
    uvm_config_db#(string)::set(null, "uvm_test_top", "golden_file", "vec/frame_out_golden.raw");

    run_test("test_basic");
  end

endmodule


```

(6) 

------

## ## 🔶 4. `uvm_env_pkg.sv` — UVM 환경 패키지

### 🎯 목적

UVM 테스트 환경의 모든 클래스(트랜잭션, 드라이버, 시퀀서, 모니터, 스코어보드, 테스트)를 **하나의 패키지**로 묶음.

### 📘 구성

| 구성요소                  | 역할                                               |
| --------------------- | ------------------------------------------------ |
| `pixel_tr.svh`        | 한 픽셀을 나타내는 트랜잭션 객체                               |
| `pixel_sequencer.svh` | 시퀀서 (sequence → driver 연결)                       |
| `pixel_driver.svh`    | TB 입력 구동기: `in_valid`, `in_pix` 신호 생성            |
| `out_monitor.svh`     | DUT의 출력(`out_pix`)을 관찰, 트랜잭션 생성 후 Scoreboard로 전달 |
| `img_scoreboard.svh`  | 골든 벡터(`frame_out_golden.raw`)와 DUT 출력 비교         |
| `input_agent.svh`     | 시퀀서+드라이버 묶음                                      |
| `env.svh`             | 전체 환경 구성 (`agent`, `monitor`, `scoreboard` 연결)   |
| `feed_raw_seq.svh`    | 입력 영상(`frame_in.raw`)을 순차적으로 보내는 시퀀스             |
| `test_basic.svh`      | 메인 테스트 클래스 (`env` 인스턴스 생성 및 실행 제어)               |

## 🔶 5. `uvm/pixel_tr.svh` — 트랜잭션 클래스

### 🎯 목적

한 번의 데이터 전송 단위를 정의 (여기선 한 픽셀).

### 📘 내용

```
class pixel_tr extends uvm_sequence_item;
  rand byte unsigned pix;
  `uvm_object_utils(pixel_tr)
endclass
```

- `pix`: 0~255 범위의 한 픽셀 값
- Sequence와 Driver가 이 객체를 주고받음

------

## 🔶 6. `uvm/pixel_driver.svh` — 드라이버

### 🎯 목적

Sequence에서 받은 트랜잭션(`pixel_tr`)을 실제 DUT 입력으로 구동

### 📘 주요 동작

1. `seq_item_port.get_next_item(tr)`으로 데이터 수신
2. `while(!vif.in_ready)` → DUT이 준비될 때까지 대기
3. `vif.in_valid=1` & `vif.in_pix=tr.pix`
4. 한 클럭 후 `vif.in_valid=0`
5. `item_done()` 호출로 완료 알림

------

## 🔶 7. `uvm/out_monitor.svh` — 모니터

### 🎯 목적

DUT 출력(`out_valid`, `out_pix`)을 관찰하고, 발생할 때마다 트랜잭션 생성하여 Scoreboard로 전달

### 📘 주요 동작

1. `@(posedge vif.rst_n)` → 리셋 해제 후 시작
2. 매 클럭마다 `if (vif.out_valid)` 검사
3. 유효할 때마다 `pixel_tr` 생성, `ap.write(tr)`로 전송

------

## 🔶 8. `uvm/img_scoreboard.svh` — 스코어보드

### 🎯 목적

DUT 출력과 **골든 데이터 파일**(`frame_out_golden.raw`)을 비교

### 📘 주요 동작

1. 시뮬 시작 시 `load_golden()`으로 파일을 읽고 `golden_q` 큐에 저장
2. Monitor에서 `write()` 호출 시, `golden_q.pop_front()`로 기대값 추출
3. `t.pix`와 비교
4. 차이 발생 시 `uvm_error("SB", ...)` 로그
5. 마지막에 PASS/FAIL 리포트 출력

------

## 🔶 9. `uvm/env.svh` — 환경 (Environment)

### 🎯 목적

UVM 컴포넌트들을 계층적으로 묶어 시퀀스→드라이버→DUT→모니터→스코어보드의 데이터 흐름을 구성

### 📘 구조

```
env
├── agent
│   ├── sequencer
│   └── driver
├── monitor
└── scoreboard
```

### 📘 동작

- `build_phase
