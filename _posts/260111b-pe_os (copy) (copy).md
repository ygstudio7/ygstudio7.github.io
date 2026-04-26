# 1. 완전 파라미터화 아키텍처 (Fully Parameterizable Architecture)

본 설계는 모든 하드웨어 구조와 메모리 매핑이 중앙 집중식 패키지(`matmul_pkg`)에 정의된 파라미터로부터 유도되는 **“Parameter-First” 설계 철학**을 구현합니다. 즉, 시스템의 모든 특성은 RTL 수정 없이 파라미터 변경만으로 제어됩니다.

## A. 핵심 하드웨어 파라미터

- **벡터화된 차원 ($NR, NC$)**
   PE 어레이의 행과 열 개수를 `NR`, `NC` 파라미터로 설정하여, 면적 제약과 목표 처리량에 따라 하드웨어 병렬성을 유연하게 조절할 수 있습니다.
  - 2×2, 4×4, 8×8 등 2의 지수승으로 자유 확장 가능: 곱셈기를 shift로 대체
  - Generate loop 기반 자동 인스턴스 생성
  - RTL 수정 없이 성능 스케일링 가능: 처리량은 **NR × NC에 비례하여 선형적으로 증가**합니다.
- **데이터 정밀도 ($DW, SW$)**
   입력 데이터(`data_t`)와 누적 결과(`sum_t`)의 비트 폭을 각각 `DW`, `SW` 파라미터로 관리합니다.
  - INT8 / INT16 / FP16 / FP32 등 다양한 정밀도 지원
  - 로직 수정 없이 정밀도 변경 가능
  - 정확도 ↔ 전력/면적 트레이드오프 조절 가능
- **시스템 상한선 (`MAX_IDX`)**
   지원 가능한 최대 행렬 차원을 `MAX_IDX`로 정의하여 시스템 전체 스케일을 사전에 규정합니다. 이를 기반으로 다음 항목들이 자동으로 설정됩니다.
  - 전체 메모리 용량(`MAX_DEPTH`)
  - 주소 비트폭(`AW`)
  - 메모리 주소 지정 범위: 대형 행렬 연산에서도 RTL 수정 없이 확장 가능합니다.

## B. 지연 시간 인식 동기화 (Latency-Aware Synchronization)

- **파이프라인 가변성**
   `AB_MEM_LATENCY`, `PE_LATENCY` 파라미터를 통해 메모리 접근 지연과 연산 파이프라인 깊이를 제어합니다.
  - 실제 SRAM/DRAM 지연 모델 반영 가능
  - 타이밍 요구사항에 따른 파이프라인 깊이 조절
  - 목표 Fmax에 맞춘 구조 최적화 가능
- **자동 지연 매칭**
   `pipe_reg` 모듈을 사용하여 제어 신호와 데이터 경로 간 지연을 자동으로 정렬합니다.
  - 파라미터 변경 시에도 기능 정확성 유지
  - 수동 지연(delay) 보정 불필요
  - 파이프라인 변경에도 강인한 구조

------

# 2. 확장성 분석 (Scalability Analysis)

확장성은 시스템이 증가하는 작업 부하를 처리하거나 성능 향상을 위해 구조를 얼마나 쉽게 확장할 수 있는지를 나타냅니다.

## A. 가속기 아키텍처 및 유연한 행렬 지원

- **가속기 인터페이스 동작**
   외부 마스터가 입력 데이터를 메모리에 기록(Write)한 후 `start` 신호를 인가하면 연산을 수행하고, 완료 후 `done` 신호를 통해 결과를 회수하는 전형적인 가속기 모델로 설계되어 시스템 통합이 용이합니다.
- **메모리 대체 가능성**
   `matmul_os_top` 모듈 내부에 입력용 A/B 메모리와 Banked C 메모리를 독립적으로 구현하여, 실제 실리콘 구현 시 표준 SRAM IP로 즉시 대체할 수 있도록 유연성을 확보했습니다.
- **런타임 타일링(Tiling) 로직**
   하드웨어 고정 크기보다 큰 행렬($M, K, N$)도 처리할 수 있도록 `cfg_t` 구조체와 타일링 로직(`mt`, `nt`)이 구현되어 있으며, 런타임에 다양한 크기의 행렬곱을 지원합니다.
- **임의 크기 지원 및 Boundary 처리**
  - `a_addr`, `b_addr` 생성 시 실제 행렬 크기(`cfg.m`, `cfg.n`)와 비교하여 유효한 영역만 접근하도록 설계되었습니다.
  - `tile_wb` 모듈에서 유효 범위 내의 데이터에 대해서만 Write Enable(`c_we`)을 활성화하여 결과 메모리에 기록합니다.

------

# 3. 연장성 분석 (Extensibility Analysis)

연장성은 기존 시스템 동작에 영향을 주지 않고 새로운 기능을 얼마나 쉽게 추가할 수 있는지를 의미합니다.

## A. 모듈형 하드웨어 설계

- **PE 레벨 추상화**
   연산 로직이 `pe_os` 모듈에 격리되어 있어 알고리즘 변경(예: Weight-Stationary) 시 해당 리프 모듈만 수정하면 전체 시스템에 반영할 수 있습니다.
- **범용 인터페이스 구조**
   `matmul_if`는 `modport`를 통해 Driver와 Monitor 역할을 명확히 구분하므로, 새로운 신호 및 프로토콜 통합이 용이합니다.

## B. 확장 가능한 UVM 테스트벤치

본 프로젝트의 검증 환경은 다양한 시나리오를 즉시 추가할 수 있도록 설계되었습니다.

- **시퀀스 확장성**
  - Randomized sequence
  - All-ones / Stress pattern
  - Ramp / Diagonal pattern
  - Directed + Random 혼합 회귀(Regression) 테스트
- **범용 컴포넌트 설계**
   Driver, Monitor, Scoreboard는 임의의 A/B 입력을 처리하도록 일반화되어 설계되었습니다.
- **팩토리 오버라이드 지원**
   모든 컴포넌트가 UVM factory에 등록되어 있어 원본 코드 수정 없이 상속 기반으로 기능을 확장할 수 있습니다.

------

# 4. 과제 요약 (Assignment Summary)

본 과제의 목표는 **파라미터화된 Systolic Array 기반 행렬곱 가속기 설계 및 검증**입니다. 주요 수행 내용은 다음과 같습니다.

- Output-Stationary 데이터플로우 구현
- NR×NC PE 어레이 구조 설계
- Banked C 메모리 구조 구현
- 런타임 구성(`cfg_t`) 기반 타일 처리
- UVM 기반 자동 검증 환경 구축

------

# 5. 시스템 개요 (System Overview)

본 시스템은 다음 주요 블록으로 구성됩니다.

- 통합 인터페이스(`matmul_if`)
- Control FSM(`IDLE/TRUN/DONE`)
- A/B 주소 생성기
- PE 어레이(`pe_array_os`)
- Banked C 메모리
- UVM 검증 환경

## 5.1 파일 구조

물리적으로는 단일 파일(`all.sv`)이지만, 논리적으로 다음과 같이 구분됩니다.

논리적으로는 다음과 같은 계층 구조로 구성됩니다.

```
project_root/
├── rtl/
│   ├── matmul_pkg.sv           # Common type definitions and global parameters package
│   ├── matmul_if.sv            # Unified DUT ↔ Testbench interface
│   ├── matmul_os_top.sv        # Top-level wrapper integrating memories and core
│   ├── matmul_os_core.sv       # Control FSM, tiling control, address generation
│   ├── pe_array_os.sv          # NR×NC PE array (generate-based instantiation)
│   ├── pe_os.sv                # Single Processing Element (MAC and accumulation)
│   ├── tile_wb.sv              # C memory write-back alignment and boundary handling
│   └── pipe_reg.sv             # Parameterized pipeline utility module
│
├── uvm/
│   ├── matmul_random_item.sv   # Transaction definition
│   ├── matmul_random_seq.sv    # Random stimulus sequence
│   ├── matmul_driver.sv        # A/B preload and start signal control
│   ├── matmul_monitor.sv       # C memory write/read observation
│   ├── matmul_scoreboard.sv    # Golden model comparison
│   ├── matmul_agent.sv         # Wrapper for driver and monitor
│   ├── matmul_env.sv           # Environment container (agent + scoreboard)
│   ├── matmul_test.sv          # Top-level UVM test
│   └── tb_top.sv               # Top-level SystemVerilog testbench
│
├── rtl.f                       # RTL file list for compilation
├── run.bat                     # Batch script to run simulation (CLI)
├── run_gui.bat                 # Batch script to run simulation (GUI mode)
└── run_win.do                  # Simulation tool command script

```

------



## 5.2 High-Level Block Diagram (Hardware Architecture)

본 설계의 상위 블록 구성은 `matmul_os_top`을 중심으로 (1) 입력 메모리 preload 경로, (2) 코어 연산 경로, (3) Banked C 메모리 저장 및 결과 읽기(read-back) 경로로 구성됩니다.

```
                         +---------------------------------------+
                         |             matmul_os_top             |
                         |                                       |
  Preload A port         |   +------------------------------+    |    Preload B port
 a_we/a_waddr/a_wdata -->|   |   A_mem (model or SRAM IP)   |    |<-- b_we/b_waddr/b_wdata
                         |   |   B_mem                      |    |
                         |   +--------------+---------------+    |
                         |                  | a_rdata[NR]        |
                         |                  | b_rdata[NC]        |
                         |                  v                    |
                         |   +------------------------------+    |
                         |   |        matmul_os_core        |----|--> done
  start, cfg ----------->|   |   (FSM: IDLE / TRUN / DONE)  |    |
                         |   |  - tiling/loop control       |    |
                         |   |  - addr gen: a_addr[NR]      |    |
                         |   |             b_addr[NC]       |    |
                         |   +---------------+--------------+    |
                         |                   |                   |
                         |           b_rdata[NC]                 |
                         |                   |                   |
                         |                   v                   |
                         |        +----------------------+       |
                         |        |      pe_array_os     |       |
                         |        |   (NR x NC generate) |       |
                         |        +----------+-----------+       |
                         |                   |                   |
                         |    c_we/c_addr/c_wdata[NR][NC]        |
                         |                   |                   |
                         |                   v                   |
                         |   +-------------------------------+   |
                         |   |     C_bank[NR][NC][DEPTH]     |   |
                         |   |  (banked write, per-PE ports) |   |
                         |   +---------------+---------------+   |
                         |                   |                   |
                         |     bank select + read mux            |
                         |  row_idx/col_idx + c_rd -> c_rdata    |
                         +---------------------------------------+
```

## 5.3 Detailed Block Diagram (Data/Control Path View)

아래는 데이터 경로(Data Path)와 제어 경로(Control Path)를 분리해 표현한 버전입니다.

```
[Control Path]
   start + cfg(M,K,N)
           |
           v
 +---------------------+
 | matmul_os_core FSM  |-----> done
 |  - tile scheduling  |
 |  - boundary check   |
 |  - addr generation  |
 +----------+----------+
            |
            | (control signals + optional pipelining via pipe_reg)
            v

[Data Path]
  +------------------+        +------------------+
  |  A_mem (NR read) |        |  B_mem (NC read) |
  +--------+---------+        +---------+--------+
           | a_rdata[NR]                | b_rdata[NC]
           +-------------+  +-----------+
                         v  v
                   +--------------+
                   |  pe_array_os |
                   |  (NR x NC)   |
                   +------+-------+
                          |
                          | c_we/c_addr/c_wdata[NR][NC]
                          v
                 +-------------------+
                 | C_bank[NR][NC]    |
                 +---------+---------+
                           |
                           | row_idx/col_idx + c_rd
                           v
                         c_rdata
```

------

# 6. DUT 인터페이스 개요

주요 신호는 다음과 같습니다.

- `clk`, `rst_n`
- `start`, `done`
- `cfg.m`, `cfg.k`, `cfg.n`
- `a_we`, `a_waddr`, `a_wdata`
- `b_we`, `b_waddr`, `b_wdata`
- `a_addr[NR]`, `a_rdata[NR]`
- `b_addr[NC]`, `b_rdata[NC]`
- `c_we[NR][NC]`
- `c_addr[NR][NC]`
- `c_wdata[NR][NC]`
- `c_rd`, `row_idx`, `col_idx`, `c_rdata`

------

# 7. RTL 모듈 설명 (RTL Module Descriptions)

본 장에서는 `all.sv`에 구현된 주요 RTL 모듈들의 역할과 설계 의도를 설명합니다. 각 모듈은 기능별로 분리되어 있으며, 확장성과 재사용성을 고려한 구조로 설계되었습니다.

## 7.1 `matmul_os_top`

- **역할**: 최상위 래퍼로 외부 시스템과 내부 코어 간 인터페이스 담당
- **주요 기능**: A/B preload, 내부 메모리 관리, Banked C 메모리 및 read-back mux, 코어 연결
- **설계 의도**: SRAM IP 대체 용이, 외부 인터페이스와 연산 코어 분리로 통합/유지보수 용이

## 7.2 `matmul_os_core`

- **역할**: FSM 기반 제어 및 주소 생성
- **주요 기능**: `IDLE→TRUN→DONE`, 타일/루프 제어, A/B 주소 생성, boundary 체크, C bank write 제어 생성
- **설계 의도**: 런타임 타일링 지원, 제어 로직 집중으로 확장 용이

## 7.3 `pe_array_os`

- **역할**: `NR×NC` PE 배열 구성
- **주요 기능**: PE 자동 인스턴스화, 데이터 전달, 파이프라인 정렬, 결과를 C bank write로 전달
- **설계 의도**: NR/NC 변경만으로 확장, 병렬 연산 구조 유지

## 7.4 `pe_os`

- **역할**: Output-Stationary PE
- **주요 기능**: 곱셈/누적, K 루프 동안 부분합 유지, 타일 종료 시 최종 결과 출력
- **설계 의도**: 리프 모듈 분리로 알고리즘 변경 영향 최소화, 향후 기능 확장 용이

## 7.5 `tile_wb`

- **역할**: 결과 write-back 정렬 및 boundary masking
- **주요 기능**: 타일 좌표 기반 주소 계산, write masking, `c_we` 생성
- **설계 의도**: 유효 결과만 기록하여 메모리 오염 방지

## 7.6 `pipe_reg`

- **역할**: 파라미터 기반 파이프라인 유틸리티
- **주요 기능**: `DW`, `N` 기반 파이프라인 구성, control/data 정렬
- **설계 의도**: 지연 모델링 및 파라미터 변경 시 자동 정렬 유지

## 7.7 `matmul_if`

- **역할**: DUT–UVM 통합 인터페이스
- **주요 기능**: 포트 집약, `modport` 기반 역할 분리, virtual interface 연결
- **설계 의도**: 검증 확장성 확보 및 디버그 신호 추가 용이

## 7.8 `matmul_pkg`

- **역할**: 공통 정의 패키지
- **포함**: `cfg_t`, `addr_t`, `data_t`, `sum_t`, `MAX_IDX`, `AW`, `MAX_DEPTH` 등
- **설계 의도**: 파라미터 중앙 관리로 Parameter-First 철학 구현

------

- # 8. UVM 검증 환경 (UVM Verification Environment)

  본 프로젝트의 검증 환경은 **UVM(Universal Verification Methodology)** 기반으로 구축되었으며, RTL 설계의 기능적 정확성을 체계적으로 검증할 수 있도록 구성되어 있습니다. 특히 다양한 행렬 크기와 타일 조합, 랜덤 입력 패턴을 효율적으로 검증할 수 있도록 **확장성과 재사용성을 최우선으로 고려한 구조**로 설계했습니다.

  ## 8.1 구성 및 역할

  검증 환경은 `matmul_if` virtual interface를 기준으로 **Sequence → Driver/Monitor → Scoreboard** 흐름으로 구성됩니다.

  - **Sequence**: 테스트 트랜잭션 생성(행렬 데이터 및 `cfg`)
  - **Driver**: A/B preload, `cfg` 설정, `start` 시퀀싱 등 입력 구동
  - **Monitor**: C bank write 활동 및 read-back 데이터 관측/수집
  - **Scoreboard**: Golden model 수행 및 DUT 결과 비교, PASS/FAIL 판정

  ```
                   +----------------------+
                   |      matmul_test     |
                   |  - config + run seq  |
                   +----------+-----------+
                              |
                              v
                   +----------------------+
                   |   matmul_random_seq  |
                   +----------+-----------+
                              |
                              v
  +----------------------+   +----------------------+   +----------------------+
  |     matmul_driver    |   |    matmul_monitor    |   |  matmul_scoreboard   |
  | - preload A/B        |   | - observe C writes   |   | - golden model       |
  | - drive start/cfg    |   | - readback sampling  |   | - compare / report   |
  +----------+-----------+   +----------+-----------+   +----------+-----------+
             |                          |                          ^
             | virtual interface        | virtual interface        |
             v                          v                          |
          +---------------------------------------------------------------+
          |                           matmul_if                           |
          +------------------------------+--------------------------------+
                                         |
                                         v
                                 +---------------+
                                 |      DUT      |
                                 | matmul_os_top |
                                 +---------------+
  ```

  ### 

  ## 8.2 컴포넌트 구조

  UVM 계층 구조는 다음과 같습니다.

  - `matmul_test`: 테스트 시작점, 환경 구성 및 시퀀스 실행 제어
  - `matmul_env`: Agent/Scoreboard 컨테이너, TLM 연결 및 설정 전달
  - `matmul_agent`: DUT I/F 담당(Driver/Monitor 포함)
  - `matmul_driver`: Transaction 기반으로 DUT 입력 구동(preload 및 `start/cfg` 제어)
  - `matmul_monitor`: DUT 출력 관측(C write 및 read-back) 후 Scoreboard로 전달
  - `matmul_scoreboard`: Golden model 계산, 비교 및 리포팅
  
  - `matmul_random_item`: 단일 테스트 트랜잭션 정의(랜덤 A/B 데이터, 랜덤 `cfg`, 옵션 등)
  - `matmul_random_seq`: 다수의 트랜잭션을 생성하여 다양한 행렬 크기 및 코너 케이스를 포함한 회귀 테스트 수행
  
  ## 8.3 확장성
  
  본 UVM 환경은 다음 확장이 용이하도록 설계되었습니다.
  
  - **추가 시퀀스**: All-zero/All-one, Stress, Directed corner, Directed+Random 혼합 회귀
  - **기능 추가**: Coverage 모델, 성능 카운터, Error injection
  - **구조적 장점**: UVM factory 등록 기반 상속/오버라이드로 원본 코드 수정 없이 확장 가능
  
  ------

  # 9. 시뮬레이션 (Simulation)

  본 장에서는 기능 검증을 위해 수행한 시뮬레이션 시나리오, 실행 흐름 및 분석 포인트를 설명합니다. 모든 시뮬레이션은 UVM 환경에서 자동 수행되며 PASS/FAIL 판정까지 자동화되어 있습니다.
  
  ## 9.1 시뮬레이션 시나리오
  
  - **랜덤 행렬곱 테스트**: 랜덤 A/B 행렬과 랜덤 `cfg(M,K,N)`를 생성하여 다양한 크기의 행렬곱을 자동 검증합니다.
  - **Boundary 테스트**: 하드웨어 타일 크기와 정합되지 않는 경우(부분 타일 포함)를 포함하여 경계 조건을 검증합니다.
  - **스트레스 테스트**: 최대 크기(`MAX_IDX`) 조건 및 반복 실행을 통해 장시간 안정성과 일관성을 확인합니다.
  
  ## 9.2 시뮬레이션 동작 흐름
  
  1. Reset 인가
  2. A/B 메모리 preload
  3. `cfg(M,K,N)` 설정
  4. `start` 신호 인가
  5. 타일 스트리밍 시작
  6. PE 연산 수행
  7. Banked C 메모리 write
  8. 결과 read-back
  9. Scoreboard 비교
  10. PASS/FAIL 출력

  ## 9.3 자동 검증 및 리포트
  
  - 모든 테스트는 **자동 PASS/FAIL 판정**을 수행합니다.
  - 실패 시 mismatch 좌표와 Golden 대비 DUT 값을 로그로 출력합니다.
  - 성공 시 테스트 이름과 함께 PASS 결과를 출력합니다.
  
  ## 9.4 GUI 파형 분석 포인트
  
  - FSM 상태(`IDLE/TRUN/DONE`)
  - `a_addr`, `b_addr` 생성 패턴과 `a_rdata`, `b_rdata` 타이밍
  - `c_we[NR][NC]` 활성화 구간
  - `c_addr`, `c_wdata` 변화
  
  이를 통해 타일 단위 연산 진행, boundary 처리, banked write 동작을 직관적으로 확인할 수 있습니다.
  
  
  
  ## 9.5 시뮬레이션 결과 요약 및 결론
  
  다양한 행렬 크기 및 boundary 조건에서 Golden model과 일치하는 결과를 확인하였으며, 파라미터 변경(NR/NC/DW 등)에 대해서도 정상 동작함을 검증했습니다.
  
  
  
  ![image-20260112003424795](assets/image-20260112003424795.png)
  
  
  
  Test 1을 확대한 waveform
  
  ![image-20260112003806199](assets/image-20260112003806199.png)
  
  
  
  
  
  ```
  # UVM_INFO uvm/tb_top.sv(68) @ 0: reporter [TOP] Starting UVM Run Test...
  # UVM_INFO @ 0: reporter [RNTST] Running test matmul_test...
  # UVM_INFO C:\intelFPGA_pro\24.2\questa_fe\verilog_src\uvm-1.2/src/base/uvm_traversal.svh(279) @ 0: reporter [UVM/COMP/NAMECHECK] This implementation of the component name checks requires DPI to be enabled
  # UVM_INFO uvm/matmul_random_seq.svh(30) @ 0: uvm_test_top.env.agt.sqr@@seq [SEQ] Generated Matrix: M=10, K=7, N=27
  # UVM_INFO uvm/matmul_driver.svh(37) @ 45000: uvm_test_top.env.agt.drv [DRV] Reset released
  # UVM_INFO uvm/matmul_monitor.svh(23) @ 45000: uvm_test_top.env.agt.mon [MON] Reset released
  # UVM_INFO uvm/matmul_driver.svh(44) @ 45000: uvm_test_top.env.agt.drv [DRV] Job 0 A loaded:
  #    [0]: 1 2 3 4 5 6 7
  #    [1]: 8 9 10 11 12 13 14
  #    [2]: 15 16 17 18 19 20 21
  #    [3]: 22 23 24 25 26 27 28
  #    [4]: 29 30 31 32 33 34 35
  #    [5]: 36 37 38 39 40 41 42
  #    [6]: 43 44 45 46 47 48 49
  #    [7]: 50 51 52 53 54 55 56
  #    [8]: 57 58 59 60 61 62 63
  #    [9]: 64 65 66 67 68 69 70
  #
  # UVM_INFO uvm/matmul_driver.svh(56) @ 745000: uvm_test_top.env.agt.drv [DRV] Job 0 B loaded:
  #    [0]: 2 2 3 1 0 2 3 0 1 3 1 2 2 1 0 2 0 0 3 0 3 3 0 0 0 1 3
  #    [1]: 1 2 3 0 2 1 1 0 2 3 0 1 3 3 1 1 3 2 3 3 2 0 3 1 0 0 2
  #    [2]: 1 1 3 2 0 3 0 3 3 0 2 3 1 3 0 2 3 1 1 2 0 0 1 0 0 2 3
  #    [3]: 3 1 0 1 2 2 0 0 1 3 2 0 1 3 3 3 3 3 1 1 1 0 3 1 3 3 3
  #    [4]: 0 2 0 2 1 1 0 1 3 1 3 0 0 3 1 1 2 0 1 1 1 3 3 1 0 0 0
  #    [5]: 3 3 0 0 1 1 3 0 1 2 3 0 3 2 2 2 1 1 1 3 0 0 3 0 1 2 1
  #    [6]: 0 0 3 1 2 1 1 2 0 1 1 1 2 1 3 1 0 0 2 2 0 3 2 1 1 0 0
  #
  # UVM_INFO uvm/matmul_driver.svh(68) @ 2635000: uvm_test_top.env.agt.drv [DRV] Job 0 start pulse
  # UVM_INFO uvm/matmul_driver.svh(77) @ 7565000: uvm_test_top.env.agt.drv [DRV] Job 0: Starting Read-back
  # UVM_INFO uvm/matmul_random_seq.svh(30) @ 10265000: uvm_test_top.env.agt.sqr@@seq [SEQ] Generated Matrix: M=29, K=13, N=11
  # UVM_INFO uvm/matmul_driver.svh(44) @ 10265000: uvm_test_top.env.agt.drv [DRV] Job 1 A loaded:
  #    [0]: 1 2 3 4 5 6 7 8 9 10 11 12 13
  #    [1]: 14 15 16 17 18 19 20 21 22 23 24 25 26
  #    [2]: 27 28 29 30 31 32 33 34 35 36 37 38 39
  #    [3]: 40 41 42 43 44 45 46 47 48 49 50 51 52
  #    [4]: 53 54 55 56 57 58 59 60 61 62 63 64 65
  #    [5]: 66 67 68 69 70 71 72 73 74 75 76 77 78
  #    [6]: 79 80 81 82 83 84 85 86 87 88 89 90 91
  #    [7]: 92 93 94 95 96 97 98 99 0 1 2 3 4
  #    [8]: 5 6 7 8 9 10 11 12 13 14 15 16 17
  #    [9]: 18 19 20 21 22 23 24 25 26 27 28 29 30
  #    [10]: 31 32 33 34 35 36 37 38 39 40 41 42 43
  #    [11]: 44 45 46 47 48 49 50 51 52 53 54 55 56
  #    [12]: 57 58 59 60 61 62 63 64 65 66 67 68 69
  #    [13]: 70 71 72 73 74 75 76 77 78 79 80 81 82
  #    [14]: 83 84 85 86 87 88 89 90 91 92 93 94 95
  #    [15]: 96 97 98 99 0 1 2 3 4 5 6 7 8
  #    [16]: 9 10 11 12 13 14 15 16 17 18 19 20 21
  #    [17]: 22 23 24 25 26 27 28 29 30 31 32 33 34
  #    [18]: 35 36 37 38 39 40 41 42 43 44 45 46 47
  #    [19]: 48 49 50 51 52 53 54 55 56 57 58 59 60
  #    [20]: 61 62 63 64 65 66 67 68 69 70 71 72 73
  #    [21]: 74 75 76 77 78 79 80 81 82 83 84 85 86
  #    [22]: 87 88 89 90 91 92 93 94 95 96 97 98 99
  #    [23]: 0 1 2 3 4 5 6 7 8 9 10 11 12
  #    [24]: 13 14 15 16 17 18 19 20 21 22 23 24 25
  #    [25]: 26 27 28 29 30 31 32 33 34 35 36 37 38
  #    [26]: 39 40 41 42 43 44 45 46 47 48 49 50 51
  #    [27]: 52 53 54 55 56 57 58 59 60 61 62 63 64
  #    [28]: 65 66 67 68 69 70 71 72 73 74 75 76 77
  #
  # UVM_INFO uvm/matmul_monitor.svh(41) @ 12365000: uvm_test_top.env.agt.mon [MON] Captured C matrix for job 0
  # UVM_INFO uvm/matmul_scoreboard.svh(26) @ 12365000: uvm_test_top.env.scb [SCB] Checking C matrix for job 0
  # UVM_INFO uvm/matmul_scoreboard.svh(54) @ 12365000: uvm_test_top.env.scb [SCB] Job 0 C PASSED:
  #    [0]: 37 41 39 28 37 39 30 28 39 45 55 20 47 62 52 46 43 25 41 53 16 39 68 18 25 31 34
  #    [1]: 107 118 123 77 93 116 86 70 116 136 139 69 131 174 122 130 127 74 125 137 65 102 173 46 60 87 118
  #    [2]: 177 195 207 126 149 193 142 112 193 227 223 118 215 286 192 214 211 123 209 221 114 165 278 74 95 143 202
  #    [3]: 247 272 291 175 205 270 198 154 270 318 307 167 299 398 262 298 295 172 293 305 163 228 383 102 130 199 286
  #    [4]: 317 349 375 224 261 347 254 196 347 409 391 216 383 510 332 382 379 221 377 389 212 291 488 130 165 255 370
  #    [5]: 387 426 459 273 317 424 310 238 424 500 475 265 467 622 402 466 463 270 461 473 261 354 593 158 200 311 454
  #    [6]: 457 503 543 322 373 501 366 280 501 591 559 314 551 734 472 550 547 319 545 557 310 417 698 186 235 367 538
  #    [7]: 527 580 627 371 429 578 422 322 578 682 643 363 635 846 542 634 631 368 629 641 359 480 803 214 270 423 622
  #    [8]: 597 657 711 420 485 655 478 364 655 773 727 412 719 958 612 718 715 417 713 725 408 543 908 242 305 479 706
  #    [9]: 667 734 795 469 541 732 534 406 732 864 811 461 803 1070 682 802 799 466 797 809 457 606 1013 270 340 535 790
  #
  # UVM_INFO uvm/matmul_driver.svh(56) @ 14035000: uvm_test_top.env.agt.drv [DRV] Job 1 B loaded:
  #    [0]: 3 0 3 1 3 1 2 1 3 2 2
  #    [1]: 1 1 1 2 2 2 1 2 0 1 1
  #    [2]: 2 2 2 1 1 0 3 3 2 2 2
  #    [3]: 1 2 1 3 2 1 2 2 3 3 1
  #    [4]: 1 3 0 1 0 0 2 0 2 1 0
  #    [5]: 2 2 0 2 2 1 2 3 1 0 3
  #    [6]: 2 0 3 3 3 1 3 3 0 2 2
  #    [7]: 2 3 3 1 3 3 0 2 3 0 1
  #    [8]: 1 2 3 3 0 2 3 0 3 1 1
  #    [9]: 2 2 0 1 0 1 3 1 2 2 3
  #    [10]: 2 3 0 1 2 2 2 1 3 1 3
  #    [11]: 2 2 0 3 2 0 2 1 1 2 2
  #    [12]: 0 0 3 3 3 1 0 3 0 2 1
  #
  # UVM_INFO uvm/matmul_driver.svh(68) @ 15465000: uvm_test_top.env.agt.drv [DRV] Job 1 start pulse
  # UVM_INFO uvm/matmul_driver.svh(77) @ 27195000: uvm_test_top.env.agt.drv [DRV] Job 1: Starting Read-back
  # UVM_INFO uvm/matmul_random_seq.svh(30) @ 30385000: uvm_test_top.env.agt.sqr@@seq [SEQ] Generated Matrix: M=22, K=3, N=7
  # UVM_INFO uvm/matmul_driver.svh(44) @ 30385000: uvm_test_top.env.agt.drv [DRV] Job 2 A loaded:
  #    [0]: 1 2 3
  #    [1]: 4 5 6
  #    [2]: 7 8 9
  #    [3]: 10 11 12
  #    [4]: 13 14 15
  #    [5]: 16 17 18
  #    [6]: 19 20 21
  #    [7]: 22 23 24
  #    [8]: 25 26 27
  #    [9]: 28 29 30
  #    [10]: 31 32 33
  #    [11]: 34 35 36
  #    [12]: 37 38 39
  #    [13]: 40 41 42
  #    [14]: 43 44 45
  #    [15]: 46 47 48
  #    [16]: 49 50 51
  #    [17]: 52 53 54
  #    [18]: 55 56 57
  #    [19]: 58 59 60
  #    [20]: 61 62 63
  #    [21]: 64 65 66
  #
  # UVM_INFO uvm/matmul_monitor.svh(41) @ 30395000: uvm_test_top.env.agt.mon [MON] Captured C matrix for job 1
  # UVM_INFO uvm/matmul_scoreboard.svh(26) @ 30395000: uvm_test_top.env.scb [SCB] Checking C matrix for job 1
  # UVM_INFO uvm/matmul_scoreboard.svh(54) @ 30395000: uvm_test_top.env.scb [SCB] Job 1 C PASSED:
  #    [0]: 137 162 126 189 160 109 167 149 153 131 163
  #    [1]: 410 448 373 514 459 304 492 435 452 378 449
  #    [2]: 683 734 620 839 758 499 817 721 751 625 735
  #    [3]: 956 1020 867 1164 1057 694 1142 1007 1050 872 1021
  #    [4]: 1229 1306 1114 1489 1356 889 1467 1293 1349 1119 1307
  #    [5]: 1502 1592 1361 1814 1655 1084 1792 1579 1648 1366 1593
  #    [6]: 1775 1878 1608 2139 1954 1279 2117 1865 1947 1613 1879
  #    [7]: 1348 1264 1255 1364 1553 874 1442 1551 1346 1060 1165
  #    [8]: 221 250 202 289 252 169 267 237 245 207 251
  #    [9]: 494 536 449 614 551 364 592 523 544 454 537
  #    [10]: 767 822 696 939 850 559 917 809 843 701 823
  #    [11]: 1040 1108 943 1264 1149 754 1242 1095 1142 948 1109
  #    [12]: 1313 1394 1190 1589 1448 949 1567 1381 1441 1195 1395
  #    [13]: 1586 1680 1437 1914 1747 1144 1892 1667 1740 1442 1681
  #    [14]: 1859 1966 1684 2239 2046 1339 2217 1953 2039 1689 1967
  #    [15]: 732 552 731 764 845 434 842 839 838 836 653
  #    [16]: 305 338 278 389 344 229 367 325 337 283 339
  #    [17]: 578 624 525 714 643 424 692 611 636 530 625
  #    [18]: 851 910 772 1039 942 619 1017 897 935 777 911
  #    [19]: 1124 1196 1019 1364 1241 814 1342 1183 1234 1024 1197
  #    [20]: 1397 1482 1266 1689 1540 1009 1667 1469 1533 1271 1483
  #    [21]: 1670 1768 1513 2014 1839 1204 1992 1755 1832 1518 1769
  #    [22]: 1943 2054 1760 2339 2138 1399 2317 2041 2131 1765 2055
  #    [23]: 116 140 107 164 137 94 142 127 130 112 141
  #    [24]: 389 426 354 489 436 289 467 413 429 359 427
  #    [25]: 662 712 601 814 735 484 792 699 728 606 713
  #    [26]: 935 998 848 1139 1034 679 1117 985 1027 853 999
  #    [27]: 1208 1284 1095 1464 1333 874 1442 1271 1326 1100 1285
  #    [28]: 1481 1570 1342 1789 1632 1069 1767 1557 1625 1347 1571
  #
  # UVM_INFO uvm/matmul_driver.svh(56) @ 31045000: uvm_test_top.env.agt.drv [DRV] Job 2 B loaded:
  #    [0]: 1 3 2 0 3 3 0
  #    [1]: 0 1 2 1 0 0 0
  #    [2]: 0 2 3 1 3 2 0
  #
  # UVM_INFO uvm/matmul_driver.svh(68) @ 31255000: uvm_test_top.env.agt.drv [DRV] Job 2 start pulse
  # UVM_INFO uvm/matmul_driver.svh(77) @ 32605000: uvm_test_top.env.agt.drv [DRV] Job 2: Starting Read-back
  # UVM_INFO uvm/matmul_monitor.svh(41) @ 34155000: uvm_test_top.env.agt.mon [MON] Captured C matrix for job 2
  # UVM_INFO uvm/matmul_scoreboard.svh(26) @ 34155000: uvm_test_top.env.scb [SCB] Checking C matrix for job 2
  # UVM_INFO uvm/matmul_scoreboard.svh(54) @ 34155000: uvm_test_top.env.scb [SCB] Job 2 C PASSED:
  #    [0]: 1 11 15 5 12 9 0
  #    [1]: 4 29 36 11 30 24 0
  #    [2]: 7 47 57 17 48 39 0
  #    [3]: 10 65 78 23 66 54 0
  #    [4]: 13 83 99 29 84 69 0
  #    [5]: 16 101 120 35 102 84 0
  #    [6]: 19 119 141 41 120 99 0
  #    [7]: 22 137 162 47 138 114 0
  #    [8]: 25 155 183 53 156 129 0
  #    [9]: 28 173 204 59 174 144 0
  #    [10]: 31 191 225 65 192 159 0
  #    [11]: 34 209 246 71 210 174 0
  #    [12]: 37 227 267 77 228 189 0
  #    [13]: 40 245 288 83 246 204 0
  #    [14]: 43 263 309 89 264 219 0
  #    [15]: 46 281 330 95 282 234 0
  #    [16]: 49 299 351 101 300 249 0
  #    [17]: 52 317 372 107 318 264 0
  #    [18]: 55 335 393 113 336 279 0
  #    [19]: 58 353 414 119 354 294 0
  #    [20]: 61 371 435 125 372 309 0
  #    [21]: 64 389 456 131 390 324 0
  #
  # UVM_INFO uvm/matmul_scoreboard.svh(62) @ 34255000: uvm_test_top.env.scb [SCB] Total jobs checked = 3, total errors = 0
  # --- UVM Report Summary ---
  #
  # ** Report counts by severity
  # UVM_INFO :   32
  # UVM_WARNING :    0
  # UVM_ERROR :    0
  # UVM_FATAL :    0
  # ** Report counts by id
  # [DRV]    13
  # [MON]     4
  # [RNTST]     1
  # [SCB]     7
  # [SEQ]     3
  # [TEST_DONE]     1
  # [TOP]     1
  # [UVM/COMP/NAMECHECK]     1
  # [UVM/RELNOTES]     1
  #
  #    Time: 34255 ns  Iteration: 68  Instance: /tb_top
  # End time: 22:04:56 on Jan 11,2026, Elapsed time: 0:00:06
  # Errors: 0, Warnings: 12
  ```
  
  
  
  ------
  
  # 10. 시뮬레이션 실행
  
  ## Windows (Questa/Modelsim)
  
  - waveform dump 자동 생성
  - PASS/FAIL 로그 출력
  
  **Non-GUI**
  
  ```
  run.bat
  ```
  
  **GUI**
  
  ```
  run_gui.bat
  ```
  
  or:
  
  ```
  vsim -do run_win.do
  ```
  
  Waveform will be in `dump.vcd`.
  
  ------
  
  # 11. 결론 및 향후 개선
  
  ## Achievements
  
  - Fully parameterized 구조 구현
  - Banked C 메모리 구조
  - 병렬 A/B read
  - UVM 기반 자동 검증 환경 구축 및 기능 검증 완료
  
  ## Future Enhancements
  
  - AXI4 인터페이스
  - DMA 엔진
  - Back-pressure 처리
  - Multi-dataflow 지원
  - Power optimization
  - Multi-core 확장