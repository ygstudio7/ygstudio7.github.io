



# uvm environment

* uvm_env 클래스는 여러 재사용 가능한 검증 콤포넌트를 포함한다. 즉, 구성을 정의하는 컨테이너 역할

* 여러 agent, scoreboard 들을 포함할 수 있가

<img src="https://www.chipverify.com/images/uvm/uvm_env_tb.svg" title="" alt="uvm_env-tb" width="682">

# why uvm components not in test class

* (-) tb안에 component를 넣을때 생기는 단점
  
  * `테스트가 특정 환경에 종속`
  
  * 테스트 작성자가 모든 구성을 알아야한다는 부담
  
  * 구조가 바뀌면 모든 테스트 파일들을 다 수정해야 한다는 부담 -> 유지보수 부담

* (+) 검증 환경을 uvm_env 기반으로 정의하고, 이것을 test에서 인스턴스로 만드는 방법을 권장
  
  * 테스트 코드와 환경 코드의 분리
  
  * 설정 (config) 변수를 두어서, agent의 설정을 제어하면 유연성 up 가능
  
  * env내에 sub-env나 block-level env가 있을 수 있으므로, topology를 미리 고려한다





# class hierarchy

* uvm_env는 uvm_component를 상속하고, 구성 요소들을 담는 컨테이너 역할

* 





# steps to create uvm env

* 사용자 정의 클래스 선언 및 팩토리 등록

```verilog
class my_env extends uvm_env;
  `uvm_component_utils (my_env)
  function new(string name="my_env", uvm_component parent = null);
    super.new(name, parent);
  endfunction
  // 이후 build_phase, connect_phase 등 구현
endclass

```



* 검증 콤포넌트 선언 및 생성

```verilog
apb_agent    m_apb_agnt;
func_cov     m_func_cov;
scbd         m_scbd;

virtual function void build_phase (uvm_phase phase);
  super.build_phase(phase);
  m_apb_agnt  = apb_agent::type_id::create("m_apb_agnt", this);
  m_func_cov  = func_cov::type_id::create("m_func_cov", this);
  m_scbd      = scbd::type_id::create("m_scbd", this);
  ...
endfunction

```



* 구성 객체가 테스트로 부터 전달되면, uvm_config_db 를 통해 가져와 사용

* 컴포넌트간 연결

```verilog
virtual function void connect_phase (uvm_phase phase);
  super.connect_phase(phase);
  // 예: 에이전트의 모니터 분석포트 → 스코어보드 분석포트 연결
endfunction

```



# uvm env example



```verilog
class my_top_env extends uvm_env;
  `uvm_component_utils (my_top_env)

  agent_apb         m_apb_agt;
  agent_wishbone    m_wb_agt;

  env_register      m_reg_env;
  env_analog        m_analog_env [2];

  scoreboard        m_scbd;

  function new (string name="my_env", uvm_component parent);
    super.new(name, parent);
  endfunction

  virtual function void build_phase (uvm_phase phase);
    super.build_phase(phase);
    m_apb_agt = agent_apb::type_id::create ("m_apb_agt", this);
    m_wb_agt  = agent_wishbone::type_id::create ("m_wb_agt", this);

    m_reg_env      = env_register::type_id::create ("m_reg_env", this);
    foreach (m_analog_env[i])
      m_analog_env[i] = env_analog::type_id::create ($sformatf("m_analog_env[%0d]", i), this);

    m_scbd = scoreboard::type_id::create ("m_scbd", this);
  endfunction
  // connect_phase etc omitted for brevity
endclass

```



## 단순한 예제

<img src="https://www.chipverify.com/images/uvm/uvm_env_block.svg" title="" alt="uvm-env-tb2" width="618">



```verilog
class my_env extends uvm_env ;

   `uvm_component_utils (my_env)

   my_agent             m_agnt0;
   my_scoreboard        m_scbd0;

   function new (string name, uvm_component parent);
      super.new (name, parent);
   endfunction : new

   virtual function void build_phase (uvm_phase phase);
      super.build_phase (phase);
      m_agnt0 = my_agent::type_id::create ("my_agent", this);
      m_scbd0 = my_scoreboard::type_id::create ("my_scoreboard", this);
   endfunction : build_phase

   virtual function void connect_phase (uvm_phase phase);
      // Connect the scoreboard with the agent
      m_agnt0.m_mon0.item_collected_port.connect (m_scbd0.data_export);
   endfunction

endclass
```



# environment reuse example

SoC의 DMA controller가 자체 검증 환경을 갖춘 독립형 장치로 이미 개별적으로 검증되었다고 가정해 보자. DMA controller가 다양한 구성의 여러 SoC에 사용되는 경우, 검증 환경은 다양한 구성을 가진 여러 시스템 레벨 테스트벤치에서도 사용될 수 있다



<img src="https://www.chipverify.com/images/uvm/uvm_env_reuse.svg" title="" alt="uvm_env_재사용" width="1341">




