# uvm sequencer

* sequence가 생성한 트랜잭션을 어떤 순서로, 언제 driver에 전달할지 관리하는 컴포넌트

* 하나의 agent내에, sequencer, driver, montor가 묶여 있고, sequencer는 트랜잭션을 dispatch하는 역할

* driver가 준비되면, pull 방식으로 sequencer에서 아이템을 가져오는 모델이 기본





# usage

<img src="https://www.chipverify.com/images/uvm/uvm_sequencer.svg" title="" alt="uvm_sequencer_env" width="708">





# how it works









# Syntax & Usage

* 기본 선언

```verilog
class uvm_sequencer#(type REQ = uvm_sequence_item, RSP = REQ)
     extends uvm_sequencer_param_base#(REQ, RSP);

```



* 에제 인스턴스

```verilog
uvm_sequencer#(my_data, data_rsp) m_seqr0;
uvm_sequencer#(my_data) m_seqr1;

```



첫번째: REQ != RSP

두번째: REQ = RSP

* 에이전트 내에서 build_phase/connect_phase에서 시퀸스와 드라이버를 생성하고 연결

```verilog
m_sequencer = uvm_sequencer#(my_data)::type_id::create("m_sequencer", this);
m_driver    = my_driver::type_id::create("m_driver", this);
...
m_driver.seq_item_port.connect( m_sequencer.seq_item_export );

```



# Example









# Custom Sequencer

기본적으로 모든 시퀀스가 uvm_sequencer 기반으로 동작하지만, 필요에 따라 특화시켜 상태변수나 기능 추가 가능

```verilog
class my_sequencer extends uvm_sequencer#(my_transaction, my_transaction);
   `uvm_component_utils(my_sequencer)
   function new(string name="m_sequencer", uvm_component parent);
      super.new(name, parent);
   endfunction
endclass

```







# Custom Sequencer Example









# m_sequencer









## Purpuse & Usage





## Example









# p_sequencer









## Purpose & Usage









## Example









# Difference btw m_sequencer and p_sequencer

* m_sequencer: 시퀸스 실행시 시스템에 자동으로 할당해주는 일반 핸들
  
  * 
    
    
    
    * p_sequencer: 타입 특화된 시퀀서를 사용하고자 할때
    
    * uvm_declare_p_sequencer(SEQUENCER) 매크로를 통해 선언하는 핸들

* 








