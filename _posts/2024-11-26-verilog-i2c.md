i2c 회로 설명

I2C(Inter-Integrated Circuit) 프로토콜은 여러 디바이스 간에 데이터를 송수신하는 데 사용되는 직렬 통신 방식입니다. 이 프로토콜은 마스터-슬레이브 구조를 기반으로 하며, 두 개의 주요 라인, **SDA (Serial Data Line)**와 **SCL (Serial Clock Line)**으로 구성됩니다.

### 주요 특징

1. **마스터-슬레이브 구조**: 하나의 마스터 디바이스가 클록 신호(SCL)를 생성하고, 마스터와 슬레이브 간의 데이터 통신을 제어합니다.
2. **두 개의 와이어**: 데이터 송수신은 SDA, 클럭 신호는 SCL을 통해 전송됩니다.
3. **다중 슬레이브 지원**: 하나의 마스터에 여러 슬레이브 디바이스를 연결할 수 있습니다. 각 슬레이브는 고유의 주소를 가지고 있습니다.
4. **비동기 통신**: 클럭 신호를 사용하여 동기화된 통신을 수행합니다.

### 작동 원리

1. **시작 조건(Start Condition)**: 마스터 디바이스가 SCL이 high 상태일 때, SDA를 high에서 low로 전환하여 통신을 시작합니다.
2. **주소 전송**: 마스터는 슬레이브 주소와 읽기/쓰기 비트를 전송합니다. 슬레이브는 이 주소를 확인하고 응답합니다.
3. **데이터 전송**: 마스터와 슬레이브는 데이터를 8비트 단위로 주고받으며, 각 바이트 후에 ACK/NACK 신호를 전송하여 전송 성공 여부를 확인합니다.
4. **정지 조건(Stop Condition)**: 마스터가 SCL이 high 상태일 때, SDA를 low에서 high로 전환하여 통신을 종료합니다.

### I2C 회로 예제

여기에 간단한 I2C 마스터 디바이스의 Verilog 코드 예제를 보여드리겠습니다.

verilog

```
module I2C_Master (
    input wire clk,
    input wire reset,
    input wire start,
    input wire [7:0] slave_address,
    input wire [7:0] data,
    output reg sda,
    output reg scl
);

reg [3:0] state;
reg [3:0] bit_index;

always @(posedge clk or posedge reset) begin
    if (reset) begin
        state <= 0;
        scl <= 1;
        sda <= 1;
        bit_index <= 0;
    end else if (start) begin
        case (state)
            0: begin
                scl <= 1;
                sda <= 0; // Start condition
                state <= 1;
            end
            1: begin
                scl <= 0;
                sda <= slave_address[7 - bit_index];
                if (bit_index == 7) begin
                    state <= 2;
                    bit_index <= 0;
                end else begin
                    bit_index <= bit_index + 1;
                end
            end
            2: begin
                scl <= 1;
                state <= 3;
            end
            3: begin
                scl <= 0;
                sda <= 1'b0; // Acknowledge bit
                state <= 4;
            end
            4: begin
                scl <= 1;
                state <= 5;
            end
            5: begin
                scl <= 0;
                sda <= data[7 - bit_index];
                if (bit_index == 7) begin
                    state <= 6;
                    bit_index <= 0;
                end else begin
                    bit_index <= bit_index + 1;
                end
            end
            6: begin
                scl <= 1;
                state <= 7;
            end
            7: begin
                scl <= 0;
                sda <= 1'b1; // Stop condition
                state <= 0;
            end
        endcase
    end
end

endmodule
```

이 예제는 I2C 마스터 디바이스의 간단한 구현을 보여줍니다. 마스터가 슬레이브 주소와 데이터를 전송하고, 통신을 시작하고 종료하는 과정을 포함합니다. 실제 구현에서는 더 복잡한 상태 기계와 오류 처리, 다양한 타이밍 요구사항을 고려해야 합니다.