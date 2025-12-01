https://www.chipverify.com/uvm/uvm-monitor

# uvm agent

* agent는 sequencer, driver, montiro를 하나로 묶은 컨테이너
* uvm 재사용성을 고려해서, active/passive, 기능 토글 등과 같은 옵션을 설정토록 함

<img src="https://www.chipverify.com/images/uvm/uvm_agent.svg" alt="img" style="zoom:150%;" />

# 유형

* active: 
  * 세 구성요소를 모두 인스턴스화 (sequencer, driver, monitor), 
  * driver로 DUT에 stimulus를 생성
* passive
  * DUT에 보낼 data가 없음 --> monitor만 인스턴스화
  * DUT 의 신호를 관찰하고, coverage 만 수행

### active/passive 확인방법

```verilog
// Assume this is inside the user-defined agent class
if (get_is_active()) begin
	// Build driver and sequencer
end
// Build monitor
```

# Class hierarchy



<img src="https://www.chipverify.com/images/uvm/uml_uvm_agent.svg" alt="uml_uvm_monitor_class_여기" style="zoom:150%;" />





# Steps to crate a uvm agent

### 1. 상속받은 uvm_agent를 factory에 등록

```verilog
// my_agent is user-given name for this class that has been derived from "uvm_agent"
class my_agent extends uvm_agent;

    // [Recommended] Makes this agent more re-usable
    `uvm_component_utils (my_agent)

    // This is standard code for all components
    function new (string name = "my_agent", uvm_component parent = null);
      super.new (name, parent);
    endfunction

    // Code for rest of the steps come here
endclass
```

### 2. agent요소 인스턴스 생성

```verilog
// Create handles to all agent components like driver, monitor and sequencer
// my_driver, my_monitor and agent_cfg are custom classes assumed to be defined
// Agents can be configured via a configuration object that can be passed in from the test
my_driver                  m_drv0;
    my_monitor                 m_mon0;
    uvm_sequencer #(my_data)   m_seqr0;
    agent_cfg                  m_agt_cfg;
```

### 3. build_phase에서 agent 요소 실제 생성: 실제 구성은 encapsulate함

```verilog
virtual function void build_phase (uvm_phase phase);

// If this UVM agent is active, then build driver, and sequencer
         if (get_is_active()) begin
            m_seqr0 = uvm_sequencer#(my_data)::type_id::create ("m_seqr0", this);
            m_drv0 = my_driver::type_id::create ("m_drv0", this);
         end

         // Both active and passive agents need a monitor
         m_mon0 = my_monitor::type_id::create ("m_mon0", this);

         //[Optional] Get any agent configuration objects from uvm_config_db
      endfunction
```

### 4. connect_phase에서 구성 요소 연결: 실제 연결은 encapsulate함

```verilog
virtual function void connect_phase (uvm_phase phase);

// Connect the driver to the sequencer if this agent is Active
         if (get_is_active())
            m_drv0.seq_item_port.connect (m_seqr0.seq_item_export);
      endfunction
```



# 역할

![img](https://www.chipverify.com/images/uvm/uvm-agent.gif)

* agent는 특정 프로토콜에 대한 시퀸싱, 드라이빙, 모니터링 기능을 하나로 묶음
* seq: transaction객체를 생성해서 driver로 보냄
* drv: 트랜잭션을 실제 인터페이스 신호로 변환 (apb_if, spi_if, axi_if 등)하여 DUT에 전달
* mon: DUT의 신호를 관찰하고, 트랜택션 객체로 변환하여 다른 컴포넌트 (scoreboard, coverage 수집기 등)에 보냄



# How to configure a UVM agent as active or passive



```verilog
// Set the configuration called "is_active" to the agent's path to mark the given agent as passive
uvm_config_db #(int) :: set (this, "path_to_agent", "is_active", UVM_PASSIVE);

// Set the configuration called "is_active" to the agent's path to mark the given agent as active
uvm_config_db #(int) :: set (this, "path_to_agent", "is_active", UVM_ACTIVE);
```



# Example



```verilog
class my_agent extends uvm_agent;
   `uvm_component_utils (my_agent)

   my_driver                  m_drv0;
   my_monitor                 m_mon0;
   uvm_sequencer #(my_data)   m_seqr0;
   agent_cfg                  m_agt_cfg;

   function new (string name = "my_agent", uvm_component parent=null);
      super.new (name, parent);
   endfunction

   // If Agent is Active, create Driver and Sequencer, else skip
   // Always create Monitor regardless of Agent's nature

   virtual function void build_phase (uvm_phase phase);
      super.build_phase (phase);

		 uvm_config_db #(agent_cfg) :: get (this, "*", "agt_cfg", m_agt_cfg);

      if (get_is_active()) begin
         m_seqr0 = uvm_sequencer#(my_data)::type_id::create ("m_seqr0", this);
         m_drv0 = my_driver::type_id::create ("m_drv0", this);
         m_drv0.vif = m_agt_cfg.vif;
      end
      m_mon0 = my_monitor::type_id::create ("m_mon0", this);

      m_mon0.vif = m_agt_cfg.vif;
   endfunction

	  // Connect Sequencer to Driver, if the agent is active

   virtual function void connect_phase (uvm_phase phase);
      if (get_is_active())
         m_drv0.seq_item_port.connect (m_seqr0.seq_item_export);
   endfunction
endclass
```



