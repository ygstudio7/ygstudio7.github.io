# 📁 권장 디렉터리 구조

```
project_root/
├─ rtl/
│  ├─ if_stream.sv 			: interface 정의
│  ├─ dut_invert.sv			: 테스트할 device
└─ uvm/                      
   ├─ uvm_env_pkg.sv         :   <-- 패키지 본체 (여기서 .svh 들을 include)
   ├─ pixel_tr.svh			: 한 픽셀을 나타내는 transaction 객체
   ├─ pixel_sequencer.svh	: sequence -> driver 연결
   ├─ pixel_driver.svh		: tb 입력 구동기, in_valid, in_pix 등 신호 생성
   ├─ out_monitor.svh		: DUT의 출력 (out_pix)를 관찰, transaction 생성후 scoreboard로 전달
   ├─ img_scoreboard.svh	: 골든 벡터 (frame_out_golden.raw) 와 DUT 출력 비교
   ├─ input_agent.svh		: sequence + driver 묶음
   ├─ env.svh				: 전체 환경 구성 (agent, monitor, scoreboard 연결)
   ├─ feed_raw_seq.svh		: 입력 영상 (frame_in.raw) 를 순차적으로 보내는 시퀀스
   └─ test_basic.svh		: 메인 test 클래스 (env 인스턴스 생성 및 실행 제어)
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



## 🔶 1. `if_stream.sv` — 인터페이스 정의

### 🎯 목적

DUT(디자인)과 테스트벤치 간에 신호를 교환하기 위한 **표준 스트리밍 인터페이스**를 정의합니다.

### 📘 주요 내용

| 항목                                | 설명                                             |
| ----------------------------------- | ------------------------------------------------ |
| `in_valid`, `in_ready`, `in_pix`    | 테스트벤치가 DUT에 입력하는 신호 (Input Stream)  |
| `out_valid`, `out_ready`, `out_pix` | DUT가 테스트벤치에 출력하는 신호 (Output Stream) |
| `modport drv`                       | 드라이버가 사용하는 방향 정의 (TB → DUT)         |
| `modport mon_o`                     | 모니터가 사용하는 방향 정의 (DUT → TB)           |

### 💡 포인트

- **하나의 인터페이스 객체(`sif`)**가 DUT과 TB를 연결함.
- `in_ready/out_ready`를 interface 내부에서 assign 하지 않고,
   DUT나 TB 쪽에서 구동해야 **다중 드라이브 충돌이 안 생김.**



## 🔶 2. `dut_invert.sv` — Device Under Test (테스트 대상)

### 🎯 목적

영상의 각 픽셀 값을 반전(invert)하는 RTL 예제.
 (예: `out_pix = 255 - in_pix`)

### 📘 주요 내용

| 항목                       | 설명                               |
| -------------------------- | ---------------------------------- |
| `PIXW`                     | 픽셀 데이터 폭(비트 수), 기본 8bit |
| `in_valid/in_ready`        | 입력 유효/준비 신호                |
| `out_valid/out_ready`      | 출력 유효/준비 신호                |
| 내부 레지스터 `v_q`, `d_q` | 파이프라인용 유효신호/데이터 저장  |

### 💡 포인트

- `in_ready=1’b1`로 고정 → 항상 입력 가능
- 한 사이클 파이프라인 구조
- 실제 영상처리 DUT을 만들 때는 여기서 convolution, filter, etc.가 들어감
- 

## 🔶 3. `uvm_env_pkg.sv` — UVM 환경 패키지

### 🎯 목적

UVM 테스트 환경의 모든 클래스(트랜잭션, 드라이버, 시퀀서, 모니터, 스코어보드, 테스트)를 **하나의 패키지**로 묶음.

### 📘 구성

| 구성요소              | 역할                                                         |
| --------------------- | ------------------------------------------------------------ |
| `pixel_tr.svh`        | 한 픽셀을 나타내는 트랜잭션 객체                             |
| `pixel_sequencer.svh` | 시퀀서 (sequence → driver 연결)                              |
| `pixel_driver.svh`    | TB 입력 구동기: `in_valid`, `in_pix` 신호 생성               |
| `out_monitor.svh`     | DUT의 출력(`out_pix`)을 관찰, 트랜잭션 생성 후 Scoreboard로 전달 |
| `img_scoreboard.svh`  | 골든 벡터(`frame_out_golden.raw`)와 DUT 출력 비교            |
| `input_agent.svh`     | 시퀀서+드라이버 묶음                                         |
| `env.svh`             | 전체 환경 구성 (`agent`, `monitor`, `scoreboard` 연결)       |
| `feed_raw_seq.svh`    | 입력 영상(`frame_in.raw`)을 순차적으로 보내는 시퀀스         |
| `test_basic.svh`      | 메인 테스트 클래스 (`env` 인스턴스 생성 및 실행 제어)        |



## 🔶 4. `tb_top.sv` — 최상위 테스트벤치

### 🎯 목적

- 클록/리셋 생성
- 인터페이스 및 DUT 인스턴스 연결
- UVM 설정 (`config_db`) 등록
- 테스트 실행 (`run_test("test_basic")`)

### 📘 주요 동작

| 구분                      | 설명                                |
| ------------------------- | ----------------------------------- |
| `clk`, `rst_n`            | TB용 기본 클럭 및 리셋 생성         |
| `if_stream sif(.*)`       | 인터페이스 생성, 자동 연결          |
| `sif.out_ready` 랜덤 토글 | DUT 백프레셔(backpressure) 발생     |
| `dut_invert` 인스턴스     | 실제 DUT 연결                       |
| `uvm_config_db::set()`    | virtual interface 및 파일 경로 등록 |
| `run_test("test_basic")`  | 지정한 테스트 시작                  |

------

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



