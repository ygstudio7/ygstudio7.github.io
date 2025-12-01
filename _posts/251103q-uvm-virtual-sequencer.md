# uvm virtual sequencer



* 드라이버와 직접 연결 X, 
* 스스로 트랜잭션을 처리하지 않음
* 주요 역할은 계층적 제어를 위해, 다른 시퀸스를 관리하고 조정
* 예들 들어, 여러 인터페이스를 가진 DUT가 있고, 각 인터페이스별 에이전트가 있고, 각각의 시퀀스가 있을때, 이들을 **<u>상위에서 조율하고 싶다</u>**면 가상 시퀸스를 사용하는게 가장 이상적이다
* 특별 class가 있는게 아니라, 일반 uvm_sequencer를 상속받아서 사용하면 된다.

<img src="https://www.chipverify.com/images/uvm/uvm_virtual_sequencer.svg" alt="uvm_virtual_sequencer" style="zoom:150%;" />



# Steps to create a virtual sequencer

### 1. Derive from uvm_sequencer

```verilog
class virtual_sequencer extends uvm_sequencer;
```



### 2. Implement Constructor and Factory Registration

```verilog
`uvm_component_utils (virtual_sequencer)

function new (string name = "my_virtual_sequencer", uvm_component parent);
	super.new (name, parent);
endfunction
```



### 3. Declare Handles for Sub-Sequencers

* 여기가 가장 큰 차이점
* 여기에 여러개의 sequencer가 들어가 있다.

```verilog
// Declare handles to other sequencers here
	apb_sequencer       m_apb_seqr;
	reg_sequencer       m_reg_seqr;
	wb_sequencer        m_wb_seqr;
	pcie_sequencer      m_pcie_seqr;

endclass
```



### 4. Instantiate and Connect

상위 수준, env에서 이들을 instance로 만들고, connect_phase에서 연결한다

```verilog
class my_env extends uvm_env;
  `uvm_component_utils (my_env)

  apb_env              m_apb_env;
  reg_env              m_reg_env;
  wb_env               m_wb_env;
  virtual_sequencer    m_vseqr;

  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);

    // Create the virtual sequencer
    m_vseqr = virtual_sequencer::type_id::create("m_vseqr", this);
  endfunction

  virtual function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);

    // Connect handles of real sequencers to the virtual sequencer
    m_vseqr.m_apb_seqr = m_apb_env.m_apb_seqr;
    m_vseqr.m_reg_seqr = m_reg_env.m_reg_seqr;
    // ...
  endfunction

endclass
```



# Guidelines for Usage

핸들이 null 인지 검사해서, 아직 active되지 않은 상태일 수 있으므로 확인해야 한ㄷ.

```verilog
class my_system_vseq extends uvm_sequence;
  `uvm_object_utils(my_system_vseq)

   virtual task body();
      ...

      // Now, check the handles to the subsequencers contained within the virtual sequencer.
      // Access these through the 'p_sequencer' handle.
      if (p_sequencer.m_apb_sqr == null) begin
         // If the APB sequencer handle is null, it indicates a configuration issue
         // (e.g., APB agent was passive, or connection in env::connect_phase failed).
         `uvm_fatal(get_full_name(), "APB Sequencer handle (p_sequencer.m_apb_sqr) is null! Check agent configuration or env::connect_phase.")

      end else begin
         `uvm_info(get_full_name(), "APB Sequencer handle is valid.", UVM_LOW)
         apb_sub_seq = apb_write_read_seq::type_id::create("apb_sub_seq", this);
         apb_sub_seq.start(p_sequencer.m_apb_sqr);
      end

     ...
   endtask
endclass
```







# Virtual Sequencer Example



```verilog
class my_virtual_sequencer extends uvm_sequencer;
	`uvm_component_utils (my_virtual_sequencer)

	function new (string name = "my_virtual_sequencer", uvm_component parent);
		super.new (name, parent);
	endfunction

	// Declare handles to other sequencers here
	apb_sequencer       m_apb_seqr;
	reg_sequencer       m_reg_seqr;
	wb_sequencer        m_wb_seqr;
	pcie_sequencer      m_pcie_seqr;
endclass
```



```verilog
class top_env extends uvm_env;
	...

	my_virtual_sequencer 	m_virt_seqr;

	virtual function void build_phase (uvm_phase phase);
		...
		m_virt_seqr = my_virtual_sequencer::type_id::create ("m_virt_seqr", this);
		...
	endfunction

	// Connect virtual sequencer handles to actual sequencers
	virtual function void connect_phase (uvm_phase phase);
		...
		m_virt_seqr.m_apb_seqr   = m_apb_agent.m_apb_seqr;
		m_virt_seqr.m_reg_seqr   = m_reg_env.m_reg_seqr;
		m_virt_seqr.m_pcie_seqr  = m_pcie_env.m_pcie_agent.m_pcie_seqr;
		...
	endfunction
endclass
```



```
----------------------------------------------------------------
CDNS-UVM-1.1d
(C) 2007-2013 멘토그래픽스 주식회사
(C) 2007-2013 Cadence Design Systems, Inc.
(C) 2006-2013 시놉시스 주식회사
(C) 2011-2013 Cypress Semiconductor Corp.
----------------------------------------------------------------
UVM_INFO @ 0: 리포터 [RNTST] 테스트 base_test 실행 중...
UVM_INFO @ 0: 리포터 [UVMTOP] UVM 테스트벤치 토폴로지:
--------------------------------------------------------------
이름 유형 크기 값
--------------------------------------------------------------
uvm_test_top 기본 테스트 - @2667
  m_top_env 탑_엔브 - @2733
    m_apb_agent apb_agent - @2780
      m_apb_drv apb_driver - @4089
        rsp_port uvm_analysis_port - @4237
        seq_item_port uvm_seq_item_pull_port - @4188
      m_apb_mon apb_monitor - @4217
      m_apb_seqr uvm_sequencer - @3512
        rsp_export uvm_analysis_export - @3569
        seq_item_export uvm_seq_item_pull_imp - @4109
        중재 대기열 배열 0 -    
        lock_queue 배열 0 -    
        num_last_reqs 정수 32 'd1  
        num_last_rsps 정수 32 'd1  
    m_spi_agent spi_agent - @2840
      m_spi_drv spi_driver - @4897
        rsp_port uvm_analysis_port - @5043
        seq_item_port uvm_seq_item_pull_port - @4995
      m_spi_mon spi_monitor - @5024
      m_spi_seqr uvm_sequencer - @4321
        rsp_export uvm_analysis_export - @4377
        seq_item_export uvm_seq_item_pull_imp - @4917
        중재 대기열 배열 0 -    
        lock_queue 배열 0 -    
        num_last_reqs 정수 32 'd1  
        num_last_rsps 정수 32 'd1  
    m_virt_seq 가상 시퀀서 - @2870
      rsp_export uvm_analysis_export - @2928
      seq_item_export uvm_seq_item_pull_imp - @3478
      중재 대기열 배열 0 -    
      lock_queue 배열 0 -    
      num_last_reqs 정수 32 'd1  
      num_last_rsps 정수 32 'd1  
    m_wb_agent wb_agent - @2810
      m_wb_drv wb_driver - @5712
        rsp_port uvm_analysis_port - @5858
        seq_item_port uvm_seq_item_pull_port - @5810
      m_wb_mon wb_monitor - @5839
      m_wb_seqr uvm_sequencer - @5136
        rsp_export uvm_analysis_export - @5192
        seq_item_export uvm_seq_item_pull_imp - @5732
        중재 대기열 배열 0 -    
        lock_queue 배열 0 -    
        num_last_reqs 정수 32 'd1  
        num_last_rsps 정수 32 'd1  
--------------------------------------------------------------

UVM_INFO ./tb/my_pkg.sv(68) @ 0: uvm_test_top.m_top_env.m_virt_seq@@m_virt_seq [VSEQ] 가상 시퀀스 시작
UVM_INFO ./tb/wb_agent.sv(63) @ 0: uvm_test_top.m_top_env.m_wb_agent.m_wb_seqr@@m_wb_reset_seq [RESET_SEQ] wb_reset_seq 시작
UVM_INFO ./tb/apb_agent.sv(52) @ 20000: uvm_test_top.m_top_env.m_apb_agent.m_apb_seqr@@m_apb_rw_seq [RW_SEQ] apb_rw_seq 시작
UVM_INFO ./tb/spi_agent.sv(74) @ 30000: uvm_test_top.m_top_env.m_spi_agent.m_spi_seqr@@m_spi_tx_seq [tx_SEQ] spi_tx_seq 시작
UVM_INFO ./tb/my_pkg.sv(75) @ 30000: uvm_test_top.m_top_env.m_virt_seq@@m_virt_seq [VSEQ] 가상 시퀀스의 끝
UVM_INFO ./tb/my_pkg.sv(108) @ 30000: uvm_test_top [SHUT] 테스트 종료 중...

--- UVM 보고서 캐처 요약 ---
```









