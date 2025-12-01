# 5) Unary Operator (SystemVerilog)

- **예**: `&a`(AND-reduction), `|a`(OR-reduction), `^a`(XOR-reduction), `~&a` 등.
- **포인트**: 버스의 집계 연산으로 빠른 조건 평가/파리티 구현.

------

# 6) Parameterized 2:1 MUX (SV)

### Point

Width인 parameter W를 사용하여, 2:1 mux이름을 mux2로 하고, 내부에 Y = sel ? A : B; 를 모듈에 구현

```verilog
module mux2 #(parameter W=8)(
  input  logic [W-1:0] a,b,
  input  logic sel,
  output logic [W-1:0] y
);
  assign y = sel ? b : a;
endmodule
```

- **포인트**: 생성자 매개변수/`typedef`로 인터페이스 일관화, 합성 예측성 유지.

------

# 7) `case` / `casez` / `case inside`

### 묻는 이유:

case 문을 제대로 쓸수 있는가?언제 case를 쓰고, 언제 casez를 써야 하나? casex를 쓰면 않되는 이유

### 1분 답변:

- 차이

  :

  - `case`: 정확히 일치.
  - `casez`: `z/?`를 와일드카드로 허용(비트마스크 매칭).
  - `case inside`: **셋/범위 포함** 매칭(수학적 집합 개념).

- **주의**: `casex`는 `x/z`를 와일드카드로 취급 → **X-masking 버그** 유발, 실무에서는 거의 금지/사용X: X 인경우를 skip하므로, 시뮬레이션, 합성시 불일치 될 수 있어서 사용하지 말아야 함

### 3분 답변

? 은 verilog에서 wide card로 (don’t care: 어떤 값이든 상관없음 )로 사용됨

casez: ?/z가 wildcard

casex: ?/z/x까지 wildcard

casex의 예: sel이 `4'b1011`, `4'b1001`, `4'b10x1`, `4'b10z1` 모두 매치: → 1.0,x,z까지 모두 매치

```verilog
reg [3:0] sel;

casex(sel)
  4'b10?1: y = 1; // 3번째 비트는 dont'care -> 0.1.x.z까지 매치
endcase
reg [3:0] sel;

casez (sel)
  4'b10?1: y = 1;   // 3번째 비트는 don't care -> 1,0만 매치
  4'b01z0: y = 2;   // 2번째 비트는 don't care -> 1,0,z 까지 매치
endcase
```

### 요점

- FSM: case 사용
- 주소 decoding: casez 사용
- casex: 사용금지

------

# 8) `wire` vs `logic/`it (SV)

### 묻는 이유:

verilog와 systemverilog 차이?, logic.bit의 차이?

### 1분 답변

- **wire**: 넷타입, **연속 할당**/모듈 간 연결, 다중 드라이버 허용.
- **logic**: 변수 타입(4-states), **단일 드라이버** 규칙(합성 관점), `always_*`/연속 할당 모두 가능(단, 다중 드라이브 금지).
- **실무 팁**: 내부 RTL은 `logic`, 토폴로지 연결은 `wire`가 가독성↑.

### 3분 답변

- wire
  - 다중 driver 허용: 버스 or-ing 가능
  - 기본값: Z
- logic
  - 단일 driver만 허용
  - 4 states: 0,1,x,z
  - 기본값 X
  - verilog reg처럼 FF 생성 가능, verilog wire처럼 port connection가능
- bit
  - 단일 드라이버만 허용
  - 2 states: 0,1 만
  - 기본값: 0
  - 주로 시뮬레이션용 (성능 최적화 목적): x,z가 없으므로

------

# 9) Blocking `=` vs Non-blocking `<=`

### 질문의도

언제 사용하나? 용도는?

### 1분 답변

- 원칙

  :

  - 순차(FF) 업데이트: **`<=`**
  - 조합 계산: **`=`**

- **이유**: 시뮬레이션/합성 동치성, 레이스/순서 의존 버그 방지.

- **예외**: 파이프라인 단계 내 임시 변수에 blocking 사용 → 조합적 평가 명확화.

### 3분 답변

## 1. 기본 정의와 차이점

| 항목               | Blocking `=`              | Non-blocking `<=`                 |
| ------------------ | ------------------------- | --------------------------------- |
| **실행 방식**      | 즉시 할당(순차 실행)      | 타임스텝 끝에 예약 후 동시에 갱신 |
| **주 용도**        | 조합 논리, 임시 변수 계산 | 순차 논리(FF 업데이트)            |
| **다중 변수 의존** | 순서에 따라 결과 달라짐   | 모든 우변 평가 후 병렬 갱신       |
| **합성 시 의미**   | FF/조합 모두 가능         | FF/조합 모두 가능                 |
| **기본 규칙**      | always_comb에서 사용      | always_ff에서 사용                |
|                    |                           |                                   |

## 2. 시뮬레이션 예제

### Blocking (`=`)

```
systemverilog
CopyEdit
always @(posedge clk) begin
  a = b;
  b = a;
end
```

- 순서대로 실행 → 첫 줄에서 a ← b
- 두 번째 줄에서 b ← a(이미 b의 값이 들어간 a) → a, b 값이 같아짐

------

### Non-blocking (`<=`)

```
systemverilog
CopyEdit
always @(posedge clk) begin
  a <= b;
  b <= a;
end
```

- 첫 줄, 두 줄 모두 “우변 평가 후” 갱신 예약
- 타임스텝 끝에 a와 b가 **서로 값 swap**

------

# 10) `function` vs `task`*

### 질문 의도

언제 사용하나?

function은 짧고 반복적인 연산을 단순화, 캡슐화할때

task는 시퀸스 기반 제어 흐름을 만들때, testbench에서 stimulus 의 시퀸스 작성시 주로 사용

### 1분 답변

- **function**: **시간 지연 X**, 출력 1개, 한 사이클 내 순수 계산
- **task**: **시간 지연 O**, 출력 n개, 프로토콜 드라이브에도 사용(테스트벤치 쪽).
- **합성**: 합성용 `function`은 순수 조합식만.

### 3분 닫변

## 1. **기본 정의**

| 항목          | function                                             | task                                                 |
| ------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **리턴값**    | 반드시 1개의 값 반환 (`return` 또는 함수명 할당)     | 리턴값 없음 (필요 시 output/inout 포트 사용)         |
| **실행 시간** | **0 시뮬레이션 시간**에서 완료 (delay, wait 불가)    | 시뮬레이션 시간 소모 가능 (delay, wait, event 가능)  |
| **호출 위치** | expression 안에서 호출 가능                          | 단독 문장으로 호출해야 함                            |
| **인자**      | input만 허용 (SV에서는 ref/output 가능하지만 제한적) | input, output, inout 모두 가능                       |
| **합성**      | 조합논리 구현 가능 (delay/이벤트 불가 시)            | 합성 가능(wait 없을 경우만), 주로 절차적 코드 구조화 |
| **사용 예시** | 계산/변환 로직                                       | 시퀀스 실행, 핸드셰이크, 복합 로직 수행              |

## 2. **코드 예시**

### function 예시

```verilog
function logic [7:0] add_byte(input logic [7:0] a, b);
    add_byte = a + b;
endfunction

logic [7:0] sum;
assign sum = add_byte(x, y);
```

- 지연 없음, 조합 연산 가능
- 식(expression) 안에서 바로 사용 가능

------

### task 예시

```verilog
task send_packet(input logic [7:0] data);
    wait (ready);
    packet <= data;
    @(posedge clk);
endtask

always_ff @(posedge clk) begin
    if (start) send_packet(payload);
end
```

- 이벤트와 대기 포함 가능
- 절차적 호출에서만 사용

------

# 11) Moore vs Mealy FSM: 거의 필수 주제*

### 질문의도

Moore FSM은 현재 상태만 의존에서 출력 생성, Mearly FSM은 현재상태와 입력을 사용

Moore는 1 clock지연으로 glitch없이 안정적, Mearly는 clock지연 없지만, 입력 변화에 따라 glitch 발생 가능

Moore는 상태수 많아질 수 있음(-), Mearly는 입력과 혼합해서 상태수 적게 가능(+)

Moore는 sequence 제어시 사용, Mearly는 handshake FSM만들때 사용

### 1분 답변

- **Moore**: 출력이 **상태에만** 의존 → 글리치 적고 타이밍 예측성↑(1-cycle latency 증가 가능).
- **Mealy**: 출력이 **상태+입력** → latency↓ 가능하나 글리치/타이밍 경계 주의.
- **현업**: 인터페이스 경계/CDC 앞은 Moore 선호, 내부 최적화는 Mealy 혼용.

### 3분 답변

## 1. 기본 정의

| 구분           | **Moore FSM**                | **Mealy FSM**                     |
| -------------- | ---------------------------- | --------------------------------- |
| 출력 결정 기준 | **현재 상태만**              | **현재 상태 + 입력**              |
| 출력 변화 시점 | 상태 전이 후 클록 엣지에서만 | 입력이 변하면 즉시 반영 가능      |
| 반응 속도      | 한 클록 늦음                 | 즉시 반응                         |
| 출력 안정성    | 안정적 (글리치 적음)         | 입력 변화에 따라 글리치 발생 가능 |
| 설계 복잡도    | 상태 수가 많아질 수 있음     | 상태 수 적게 가능                 |
| 타이밍 제어    | 예측 쉬움                    | 타이밍 제어 어려움                |

------

## 2. 코드 예시

### Moore FSM 예시

```verilog
always_ff @(posedge clk or posedge rst) begin
    if (rst) state <= IDLE;
    else     state <= next_state;
end

always_comb begin
    case (state)
        IDLE:    out = 0;
        ACTIVE:  out = 1;
    endcase
end
```

- **출력은 오직 state에만 의존**
- 입력 변화는 다음 상태 전이 후 반영됨

------

### Mealy FSM 예시

```verilog
always_ff @(posedge clk or posedge rst) begin
    if (rst) state <= IDLE;
    else     state <= next_state;
end

always_comb begin
    case (state)
        IDLE:    out = (in) ? 1 : 0; // 입력 즉시 반영
        ACTIVE:  out = (in) ? 0 : 1;
    endcase
end
```

- **출력이 입력에도 의존**
- 입력 변화가 클록 사이클 중간에 바로 반영될 수 있음