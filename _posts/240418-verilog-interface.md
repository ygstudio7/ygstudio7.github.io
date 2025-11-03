# 1 

시스템베릴에서 **인터페이스를 연결하는 방법**은 다음과 같습니다:

1. **인터페이스 정의**:

   - 먼저, 인터페이스를 정의합니다. 인터페이스는 관련된 신호를 묶어서 블록으로 캡슐화하는 방법입니다. 인터페이스 내에서 신호를 선언하고 `endinterface`로 끝납니다. 예를 들어 APB 버스 프로토콜 신호를 아래와 같이 인터페이스에 묶을 수 있습니다:

   ```systemverilog
   interface apb_if (input pclk);
       logic [31:0] paddr;
       logic [31:0] pwdata;
       logic [31:0] prdata;
       logic penable;
       logic pwrite;
       logic psel;
   endinterface
   ```

   

2. **인터페이스 인스턴스화**:

   - DUT가 인터페이스를 사용하도록 인스턴스화해야 합니다. 이를 통해 DUT와 인터페이스가 연결됩니다. 예를 들어:

   ```systemverilog
   module dut (apb_if bus_if);
       // DUT 내에서 인터페이스 신호 사용
       // ...
   endmodule
   ```

   

3. **상위 모듈에서 인터페이스 객체 생성**:

   - 상위 테스트벤치 모듈에서 인터페이스 객체를 생성하고 DUT에 전달해야 합니다. 올바른 `modport`가 DUT에 할당되었는지 확인해야 합니다:

   ```systemverilog
   module tb_top;
       bit clk;
       always #10 clk = ~clk;
   
       apb_if bus_if (clk);
       dut dut0 (bus_if);
   
       // 테스트 시나리오 설정
       // ...
   endmodule
   ```

   

인터페이스를 사용하면 작업량을 줄이고 특정 블록에서 트랜잭션을 모니터링하고 기록할 수 있습니다. [또한 인터페이스를 통해 설계에 연결하는 것이 더 쉬워집니다](https://www.doulos.com/knowhow/systemverilog/systemverilog-tutorials/systemverilog-interfaces-tutorial/)[1](https://www.doulos.com/knowhow/systemverilog/systemverilog-tutorials/systemverilog-interfaces-tutorial/)[2](https://m.blog.naver.com/doksg/222135709975)[3](https://www.chipverify.com/systemverilog/systemverilog-interface).

# 1

시스템베릴에서 인터페이스를 한 줄로 할당하는 방법은 다음과 같습니다:

1. 먼저 인터페이스를 선언합니다. 인터페이스는 관련된 신호를 묶어서 블록으로 캡슐화하는 방법입니다. 인터페이스 내에서 신호를 선언하고 `endinterface`로 끝납니다. 예를 들어 APB 버스 프로토콜 신호를 아래와 같이 인터페이스에 묶을 수 있습니다:

   ```systemverilog
   interface apb_if (input pclk);
       logic [31:0] paddr;
       logic [31:0] pwdata;
       logic [31:0] prdata;
       logic penable;
       logic pwrite;
       logic psel;
   endinterface
   ```

   

2. 인터페이스 내에서 신호를 `logic`으로 선언합니다. `logic`은 `assign` 문이나 절차 블록에서 해당 유형의 신호를 구동할 수 있도록 해줍니다. 또한 DUT에 연결된 신호는 X/Z 값을 캐치할 수 있도록 4-state를 지원해야 합니다.

3. 인터페이스 신호의 방향을 정의하기 위해 `modport`를 사용합니다. 다른 컴포넌트에 다른 입력-출력 방향을 정의할 수 있도록 `modport`를 다르게 정의할 수 있습니다. 예를 들어:

   ```systemverilog
   interface myBus (input clk);
       logic [7:0] data;
       logic enable;
       modport TB (input data, clk, output enable);
       modport DUT (output data, input enable, clk);
   endinterface
   ```

   

4. DUT가 인스턴스화된 상위 테스트벤치 모듈에서 인터페이스 객체를 생성하고 DUT에 전달해야 합니다. 올바른 `modport`가 DUT에 할당되었는지 확인해야 합니다:

   ```systemverilog
   module dut (myBus busIf);
       always @(posedge busIf.clk)
           if (busIf.enable)
               busIf.data <= busIf.data + 1;
           else
               busIf.data <= 0;
   endmodule
   
   module tb_top;
       bit clk;
       always #10 clk = ~clk;
   
       myBus busIf (clk);
       dut dut0 (busIf.DUT);
   
       initial begin
           busIf.enable <= 0;
           #10 busIf.enable <= 1;
           #40 busIf.enable <= 0;
           #20 busIf.enable <= 1;
           #100 $finish;
       end
   endmodule
   ```

   

인터페이스를 사용하면 작업량을 줄이고 특정 블록에서 트랜잭션을 모니터링하고 기록할 수 있습니다. [또한 인터페이스를 통해 설계에 연결하는 것이 더 쉬워집니다.](https://www.chipverify.com/systemverilog/systemverilog-interface)[1](https://www.chipverify.com/systemverilog/systemverilog-interface)[2](https://learn.verificationstudio.com/tutorial/systemverilog-tutorial/interfaces)



