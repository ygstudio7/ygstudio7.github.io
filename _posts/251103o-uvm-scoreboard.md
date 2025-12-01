# uvm scoreboard

scorebard는 검증 컴포넌트로서, 체커를 포함하고, DUT 의 기능을 검증하는 역할

예를들어, RW 용 register가 있으면, write 시, scoreboard기 이 패킷을 받아 예상값을 저정하고, read시 scorebard가 이 패킷을 받아 실제 읽은 값과 예상값을 비교하는 방식이다

# reference model

scoreboard는 데이터 수신후 

- 직접 계산해서 예상을 만들 수도 있고, 
- reference model에 값을 넘겨서 예상값을 받아올 수 도 있다. 이 모델은 설계의 기능을 모방하도록 설계된다.

최종적으로, expected와 actual output을 비교해야 맞는지 판단한다

<img src="https://www.chipverify.com/images/uvm/uvm_scoreboard.svg" alt="uvm 스코어보드" style="zoom:150%;" />

# Steps to create a uvm scoreboard

### 1. uvm_scoreboard를 상속받은 클래서를 생성하고, 팩토리에 등록

```verilog
// my_scoreboard is user-given name for this class that has been derived from "uvm_scoreboard"
class my_scoreboard extends uvm_scoreboard;

    // [Recommended] Makes this scoreboard more re-usable
    `uvm_component_utils (my_scoreboard)

    // This is standard code for all components
    function new (string name = "my_scoreboard", uvm_component parent = null);
      super.new (name, parent);
    endfunction

    // Code for rest of the steps come here
endclass
```

### 2. 다른 구성요소에서 트랜잭션을 수신할 수 있도록 TLM 분석 포트 선언 및 build_phase에서 인스턴스화

```verilog
// Step2: Declare and create a TLM Analysis Port to receive data objects from other TB components
uvm_analysis_imp #(apb_pkt, my_scoreboard) ap_imp;

// Instantiate the analysis port, because afterall, its a class object
function void build_phase (uvm_phase phase);
	ap_imp = new ("ap_imp", this);
endfunction
```

### 3. 분석 포트에서 데이터 수신시 수행할 작업을 정의

```verilog
// Step3: Define action to be taken when a packet is received via the declared analysis port
virtual function void write (apb_pkt data);
	// What should be done with the data packet received comes here - let's display it
	`uvm_info ("write", $sformatf("Data received = 0x%0h", data), UVM_MEDIUM)
endfunction
```

### 4. 점검 수행

```verilog
// Step4: [Optional] Perform any remaining comparisons or checks before end of simulation
virtual function void check_phase (uvm_phase phase);
	...
endfunction
```

### 5. 스코어 보드의 분석 포트를 환경의 다른 구성요소와 연결

```verilog
class my_env extends uvm_env;
	...

	// Step5: Connect the analysis port of the scoreboard with the monitor so that
	// the scoreboard gets data whenever monitor broadcasts the data.
	virtual function void connect_phase (uvm_phase phase);
		super.connect_phase (phase);
		m_apb_agent.m_apb_mon.analysis_port.connect (m_scbd.ap_imp);
	endfunction
endclass
```







# Example

페이지에는 간단한 예제가 포함되어 있습니다. 예컨대:

- 클래스 `my_scoreboard`가 `uvm_scoreboard`를 상속
- `uvm_analysis_imp #(switch_item, scoreboard) m_analysis_imp;` 선언
- `build_phase`에서 `m_analysis_imp = new("m_analysis_imp", this);`
- `write(switch_item item)` 메소드 내부에서 주소(addr) 범위에 따라 분기하고 예상값 vs 실제값 비교 및 `uvm_error` 또는 `uvm_info` 출력. [ChipVerify+1](https://www.chipverify.com/uvm/uvm-testbench-example-2?utm_source=chatgpt.com)

```verilog
class my_scoreboard extends uvm_scoreboard;
    `uvm_component_utils(my_scoreboard)
    
    function new(string name = "my_scoreboard", uvm_component parent);
        super.new(name, parent);
    endfunction
    
    uvm_analysis_imp #(apb_pkt, my_scoreboard) ap_imp;
    
    function void build_phase (uvm_phase phase);
        ap_imp = new("ap_imp", this);
    endfunction
    
    virtual function void write(apb_pkt data);
        `uvm_info("write", $sformatf("Data received = 0x%0h", data), UVM_MEDIUM)
    endfunction
    
    virtual task run_phase(uvm_phase phase);
        ...
    endtask
    
    virtual function void check_phase (uvm_phase phase);
        ...
    endfunction
    
endclass

```





# How to connect analysis ports 

monitor와 scoreboard의 연결은 env의 connect_phase에서 이뤄진다. 

scoreboard를 정의한 후에는 환경에서 스코어보드를 인스턴스화하고, 적절한 분석 포트에 연결해야 한다.

```verilog
class my_env extends uvm_env;
    `uvm_component_utils (my_env)
    
    function new(string name = "my_env", uvm_component parent);
        super.new(name, parent);
    endfunction
    
    my_scoreboard m_scbd;
    
    virtual function void build_phase(uvm_phase phase);
        super.build_phase(phase);
        
        m_scbd = my_scoreboard::type_id::create("m_scbd"m this);
    endfunction
    
    virtual function void connect_phase(uvm_phase phase);
        super.connect_phase(phase);
        
        m_apb_agent.m_apb_mon.analysis_port.connect(m_scbd.ap_imp);
    endfunction
endclass
```





