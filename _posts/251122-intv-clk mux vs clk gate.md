# Glitch-free clock mux vs clock gating

Clock mux는 여러 클록 중 하나를 선택하는 기능이고,
clock gating은 동일한 클록을 전력 절약을 위해 켜고 끄는 기능입니다.
두 회로의 목적과 요구되는 timing/CDC 제약이 완전히 다릅니다.

# 비교

| 기능             | 목적           | 동작                  | Glitch 위험                 | 용도                          |
| ---------------- | -------------- | --------------------- | --------------------------- | ----------------------------- |
| **Clock Mux**    | 클록 소스 전환 | clk0 ↔ clk1 switching | 높음 → glitch-free mux 필요 | DVFS, failover, PLL switching |
| **Clock Gating** | 전력 절감      | 클록 ON/OFF           | 낮음 (내부 latch)           | power saving, sleep mode      |




| 상황                               | 권장 방식                     | 이유                                              |
| ---------------------------------- | ----------------------------- | ------------------------------------------------- |
| 고속 SoC 설계, PLL switching, DVFS | **glitch-free clock mux IP**  | 안정성, formal 검증, PNR 적합                     |
| Async clock switching              | **glitch-free mux**           | select sync 필수                                  |
| Low-power mode, domain ON/OFF      | **clock gate switching 가능** | domain off/on만 필요, glitch-free mux 필요도 낮음 |
| SW controlled clock source 변경    | **glitch-free mux**           | 안정성 필수                                       |
| 아주 단순한 dual-clock 선택        | **clock gate 방식도 OK**      | handshake만 잘하면 됨                             |

# Glitch-Free Clock Switching (전용 mux 기반)

**두 개의 클록 중 원하는 클록으로 안전하게 전환**하기 위한 구조.
 핵심은 **클록 에지 중간에서 mux가 바뀌면 glitch(짧은 펄스)가 발생**하므로, 이를 방지하기 위해 **두 클록 모두 LOW일 때만 mux select를 바꾼다.**

### ✔️ 기본 원리

- 일반 mux:
   `clk_out = sel ? clk1 : clk0`  → sel 바뀌는 순간 두 클록 위상이 다르면 glitch 발생.
- Glitch-free mux:
  - 두 클록이 **공통 safe point(보통 both low)**일 때만 mux switch
  - 내부적으로 **AND/OR 구조 + FF sync**를 사용함
  - (sel 신호를 **양쪽 클록 domain에서 latch**)

### ✔️ 장점

- **완전한 glitch-free**
- **고속에서도 안전**
- Clock tree synthesis 에서도 문제 없음
- 리던던트 clock failover 구조에 최적화됨

### ✔️ 단점

- 구조가 상대적으로 복잡
- Select는 반드시 **double-synchronizer** 필요
- 두 클록이 **동시에 low가 되는 순간까지 wait**해야 하므로 전환 latency가 clock gate 방식보다 약간 길 수 있음



# SDC for clock mux


문제는 STA가 clock path를 계산할 때:

- clk0와 clk1의 주파수가 다를 수 있고
- 서로 phase 관계가 없고
- 서로 asynchronous일 수 있고
- mux 출력은 어느 clock일지 “조건에 따라” 달라짐

그래서 “SDC에서 이 구조를 정확히 알려줘야”  STA가 올바른 clock tree, timing path, CDC를 판단할 수 있다.

정리: 
1) input clocks 정의
2) input clocks 를 asynchronous group 으로 묶기
3) mux output에 generated clock 2개 적용 (-add)
4) 모드별 case analysis 필요 시 적용

Clock Mux는 **둘 이상의 clock source 중 하나를 선택(switch)**하는 구조다.



------

## 2. 기본 구조 예시

```
clk0 -----\
           MUX ─── clk_out → logic
clk1 -----/
         sel
```

------

## 3. Clock Mux SDC 전체 구조 (필수 4단계)

### 1) 입력 클록 정의

```
create_clock -name CLK0 -period 2.000 [get_ports clk0]
create_clock -name CLK1 -period 4.000 [get_ports clk1]
```

------

### 2) 입력 두 클록은 서로 async 그룹

두 클록이 서로 어떤 phase relationship도 없다는 것을 명시.

```
set_clock_groups -asynchronous \
  -group {CLK0} \
  -group {CLK1}
```

➡️ 이게 없으면 STA가 clk0→clk1 cross path를 잡아서 timing fail 발생.

------

### 3) mux output에 두 클록 기반의 generated clock을 **겹쳐서 정의 (-add)**

**출력 clk_out은 상황에 따라 다음 둘 중 하나일 수 있다:**

- clk0가 선택 → CLK_SW0
- clk1가 선택 → CLK_SW1

이를 정확히 모델링하기 위해 다음처럼 **두 개의 generated clock을 하나의 pin에 덮는다**:

```
create_generated_clock -name CLK_SW0 \
  -source [get_ports clk0] \
  -divide_by 1 \
  [get_pins u_mux/clk_out]

create_generated_clock -name CLK_SW1 \
  -source [get_ports clk1] \
  -divide_by 1 \
  -add \
  [get_pins u_mux/clk_out]
```

### 📌 왜 -add가 필요한가?

- STA는 *하나의 clock pin에 여러 clock을 overlay* 할 수 있어야 한다.
- clock mux는 “둘 중 하나”의 clock이 output으로 나올 수 있기 때문.

------

### 4) 모드별 분석을 위한 case analysis (선택적)

Switching이 dynamic이라면 없어도 되지만, 정적 모드 STA(Mode 0 / Mode 1)를 분리 분석할 때 필요.

**Mode0 (mux가 항상 clk0 선택)**

```
set_case_analysis 0 [get_pins u_mux/sel]
```

**Mode1 (mux가 항상 clk1 선택)**

```
set_case_analysis 1 [get_pins u_mux/sel]
```

이렇게 하면:

- Mode0에서 STA는 CLK_SW0만 활성 clock으로 봄
- Mode1에서는 CLK_SW1만 활성 clock으로 봄

그래서 불필요한 pessimism이 제거됨.

------

## 옵션: mux 내부에 latch sync logic이 있다면?

sel이 clock domain crossing이므로  해당 sync flip-flop 경로는 false path로 처리:

```
set_false_path -to [get_pins u_mux/sel_sync*/D]
```