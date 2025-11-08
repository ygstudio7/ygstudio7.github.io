[UVM Monitor [uvm_monitor]](https://www.chipverify.com/uvm/uvm-monitor)



# monitor

* monitor는 dut 쪽 인터페이스에서 신호 및 버스 활동을 관찰 후, 트랜잭션 레벨 게체로 변환해 상위 컴포넌트 (scorebard 등)로 전달

* 신호/버스 활동 수집: 

* 프로토콜 검사 및 coverage 연계
  
  * 기본적인 프로토콜 규칙 검사 기능 포함
  
  * functional coverage 수집 기능 포함

* TLM 분석 포트로 데이터 방출





# class hierarchy









# steps to create a uvm monotor

1. 선언 및 상속

```verilog
class my_monitor extends uvm_monitor;
   `uvm_component_utils(my_monitor)
   // …
endclass
```



2. 변수/포트 선언

```verilog
virtual my_if vif;
uvm_analysis_port #(my_data) mon_analysis_port;
bit enable_check = 1;
bit enable_coverage = 1;

```



* vif: DUT의 인터페이스를 가리키게 됨

* mon_analysis_port: 수집된 트랜잭션 객체를 송출하는 통로

* enable_check, enable_coverage: 모니터 기능을 켜고 끌 수 있는 제어 변수



3. build_phase 구현

```verilog

virtual function void build_phase(uvm_phase phase);
   super.build_phase(phase);
   mon_analysis_port = new("mon_analysis_port", this);
   if (!uvm_config_db #(virtual my_if)::get(this, "", "vif", vif)) begin
      `uvm_error(get_type_name(), "DUT interface not found")
   end
endfunction

```

* 분석 포트 인스턴스 생성

* uvm_config_db에서 vif 획득



4. run_phase 구현

```verilog
virtual task run_phase(uvm_phase phase);
   my_data data_obj = my_data::type_id::create("data_obj", this);
   forever begin
      @(/* some event on vif indicating valid data */);
      data_obj.data = vif.data;
      data_obj.addr = vif.addr;
      
      if (enable_check) check_protocol();
      if (enable_coverage) data_obj.cg_trans.sample();
      
      mon_analysis_port.write(data_obj);
   end
endtask

```

* 이벤트를 기다린후 (예: `@posedge vif.valid`),

* 신호값을 읽어서 (vif), 트랜잭션 객체로 전환후 (data_obj)

* if(): 프로토콜 검사 및 커버리지 샘플링

* 분석포트로 트랜잭션 객체 전달



# uvm monitor example









# recommended practice










