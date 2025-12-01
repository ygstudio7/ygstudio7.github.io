# 4) Gray Code

### 질문 의도

언제 사용하나?  비동기 환경 (CDC)에서 비트 전이를 최소화하면, CDC에서 안정성 증가

주로 Async FIFO에서 address pointer를 전달시 사용

### 1분 답변

- **왜 쓰나**: CDC에서 **포인터 다중비트 변경 위험**을 1-bit 변화로 제한 → 잘못된 샘플링 확률 ↓.
- **예**: Async FIFO의 read/write 포인터 전송.

### 3분 답변

gray code: 인접한 두 수가 1 bit만 다르게 표현되는 코드

예: 3 bits

예: 3비트 Gray Code 순서

| Decimal | Binary | Gray |      |
| ------- | ------ | ---- | ---- |
| 0       | 000    | 000  |      |
| 1       | 001    | 001  |      |
| 2       | 010    | 011  |      |
| 3       | 011    | 010  |      |
| 4       | 100    | 110  |      |
| 5       | 101    | 111  |      |
| 6       | 110    | 101  |      |
| 7       | 111    | 100  |      |

- binary counter: 011b → 100b로 모두 변경
- gray counter: 010b → 110b로 1bit 만 변경 → multibit 전달시 glitch 최소화

binary → gray 변환 공식

```jsx
gray[N-1] = bin[N-1];  // MSB 동일

gray[i] = bin[i+1] ^ bin[i];  // 나머지는 인접 비트 xor
```

gray → binary 변환 공식

```jsx
bin[N-1] = gray[N-1];
bin[i] = bin[i+1] ^ gray[i];
```

## 4. 예제: Async FIFO Pointer Sync

wptr을 rclk으로 넘길때 3단계로 진행: wptr_bin → wptr_gray → 2 FFs @rclk → wptr_gray_rclk → wptr_bin_rclk: 2clock cycles지연 @rclk

```verilog
// Write domain에서 binary pointer -> gray 변환 // wptr_bin -> wptr_gray
assign wr_ptr_gray = (wr_ptr_bin >> 1) ^ wr_ptr_bin; 

// Read domain에서 2-stage sync 후 binary로 변환 // wptr_gray -> 2 FFs -> wptr_gray_rclk
always_ff @(posedge rd_clk) begin 
    sync1 <= wr_ptr_gray;
    sync2 <= sync1;
end

assign wr_ptr_bin_sync = gray2bin(sync2); // wptr_gray_rclk -> wptr_bin_rclk
```



# 14) 동기식 vs 비동기식 FIFO

- **동기식**: 단일 클록, 단순/고속.
- **비동기식**: 서로 다른 클록; **Gray 포인터+2-FF** 동기화, full/empty 계산은 **동기화된 상대 포인터** 기반. 타이밍 클로저는 쉬우나 CDC 정합/정적검증 필수.
- 

# 13) Async 이벤트 전달 시 “최소 클록”

### 질문 의도

async event 전달 방법을 알고 있나?

### 1분 답변

- **단일 비트 플래그**를 2-FF sync로 수신: **수신 도메인 기준 최소 2클록** 지연(표본→안정화).
- **토글/펄스**는 **펄스 스트레치**(≥2 dest clk) 또는 토글-캡쳐 방식 권장.
- **Async FIFO**: 포인터 2-FF 동기화 → **풀/엠프티 플래그 안정**까지 수신 도메인 **2~3클록** 여유 필요.

### 3분 답변

1. tx pulse width ≥ (1.5~2) x rx pulse width인 경우, 2 FFs로 전달 가능
2. otherwise, toggle flop 방식을 사용
   1. 펄스 대신 토글 신호를 전송하고, 수신측에서 xor변화를 검출

## 2. **토글 방식 (Toggle Synchronization)**

**개념**

- 이벤트가 발생할 때마다 **비트를 토글**하고, 수신 도메인에서는 **이전 값과 XOR**하여 이벤트 검출
- 펄스 폭 제약이 없어 CDC 안정성이 높음

**예시 코드**

**송신 도메인: toggle해서 보낸다**

```verilog
always_ff @(posedge clk_src or posedge rst_src) begin
    if (rst_src)
        toggle_bit <= 0;
    else if (event_in)
        toggle_bit <= ~toggle_bit;
end
```

**수신 도메인: 3FFs 후, 마지막 2 FFs를 xor한다.**

```verilog
logic sync1, sync2, toggle_prev;

always_ff @(posedge clk_dst or posedge rst_dst) begin
    if (rst_dst) begin
        {sync2, sync1} <= 0;
        toggle_prev <= 0;
    end else begin
        sync1 <= toggle_bit;  // stage 1 sync
        sync2 <= sync1;       // stage 2 sync
        toggle_prev <= sync2; // 이전 값 저장
    end
end

assign event_detected = (sync2 ^ toggle_prev); // XOR로 변화 검출
```

- (+) 이벤트가 1클록보다 짧아도 안전하게 검출 가능
- **(-)** 동일 이벤트가 너무 빠르게 연속 발생하면 놓칠 수 있음 (수신 도메인에서 1번 이상 샘플링 전에 다시 토글되면 문제)

