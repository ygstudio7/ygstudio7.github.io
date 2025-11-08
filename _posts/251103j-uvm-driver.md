# uvm driver

* uvm_sequencer는 트랜잭션을 다루고, uvm driver는 신호를 구동해서 DUT에 적용하는 역할을 한다.

* uvm_driver #(REQ, RSP)  형태로 파라미터화 되어 있고, 요청 트랜잭션 타입(REQ),  응답 트랜잭션 타입 (RSP)을 정의

* 드라이버는 시퀀서로부터 트랜잭션을 받고, 내부에서 트랜잭션 정보를 interface 로 매핑해서 구동한 뒤, 필요하다면 응답을 시퀀서로 되돌려준다 

# class hierarchy

<img src="https://www.chipverify.com/images/uvm/uml_uvm_driver.svg" title="" alt="uml_uvm_driver_class_hier" width="521">

# steps to create a uvm driver

<img title="" src="https://www.chipverify.com/images/uvm/uvm_driver.svg" alt="uvm_driver-env" width="770">

1. 사용자 정의 클래스 선언 및 uvm_component_utils 매크로 사용하여 팩토리 등록

```verilog
class my_driver extends uvm_driver #(my_item);
  `uvm_component_utils(my_driver)
  function new(string name = "my_driver", uvm_component parent = null);
    super.new(name, parent);
  endfunction
  // 이후 build_phase, run_phase 구현
endclass
```

2. 인터페이스 핸들 (virtual interface) 선언 및 build_phase에서 획득

[TODO] 인터페이스 핸들을 통해 DUT와 의 실제 연결이 이뤄진다고 하는데,... 여기 부분 코드들이 잘 이해가 안된다.
이건 "vif"라는 키를 사용하여, 이미 등록된 실제 인터페이스를 vif로 가져오는 것이고, 이를 vif를 이용해서 나중에 run_phase같은데서, vif 신호를 구동한다.

```verilog
virtual if_name vif;
virtual function void build_phase(uvm_phase phase);
  super.build_phase(phase);
  if (!uvm_config_db#(virtual if_name)::get(this, "", "vif", vif)) begin
    `uvm_fatal(get_type_name(), "Didn't get handle to virtual interface if_name")
  end
endfunction
```



3. run_phase안에 드라이버의 주요 동작 구현

```verilog
virtual task run_phase(uvm_phase phase);
  super.run_phase(phase);
  forever begin
    req_item_type req;
    seq_item_port.get_next_item(req);
    // 신호 구동 코드: vif.some_signal <= req.some_field;
    seq_item_port.item_done();
  end
endtask
```

* seq_item_port.get_next_item()으로 sequencer에서 item을 받기

* 신호 구동후, item_done()호출해서 시퀸서에게 완료를 알리기



# uvm driver-sequencer handshake

* get_next_item: 다음 항목 가져오기 

* item_done: 완료를 알리기, get_next_item후에 호출되어야 한다?









# how are driver/sequencer API methods used?

1. get_next_item 다음에, item_done()



```verilog
virtual task run_phase (uvm_phase phase);
	my_data req_item;

	forever begin
		// 1. Get next item from the sequencer
		seq_item_port.get_next_item (req_item);

		// 2. Drive signals to the interface
		@(posedge vif.clk);
		vif.en <= 1;
		// Drive remaining signals, put write data/get read data

		// 3. Tell the sequence that driver has finished current item
		seq_item_port.item_done();
	end
```

2. get() 다음에 put()

```verilog
virtual task run_phase (uvm_phase phase);
	my_data req_item;

	forever begin
		// 1. finish_item in sequence is unblocked
		seq_item_port.get (req_item);

		// 2. Drive signals to the interface
		@(posedge vif.clk);
		vif.en = 1;
		// Drive remaining signals

		// 3. Finish item
		seq_item_port.put (rsp_item);
	end
endtask
```





# uvm driver example



```verilog
class my_driver extends uvm_driver #(my_data);
  `uvm_component_utils (my_driver)

   virtual  dut_if   vif;

   function new (string name, uvm_component parent);
      super.new (name, parent);
   endfunction

   virtual function void build_phase (uvm_phase phase);
      super.build_phase (phase);
      if (! uvm_config_db #(virtual dut_if) :: get (this, "", "vif", vif)) begin
         `uvm_fatal (get_type_name (), "Didn't get handle to virtual interface dut_if")
      end
   endfunction

   task run_phase (uvm_phase phase);
      my_data data_obj;
      super.run_phase (phase);

      forever begin
         `uvm_info (get_type_name (), $sformatf ("Waiting for data from sequencer"), UVM_MEDIUM)
         seq_item_port.get_next_item (data_obj);
         drive_item (data_obj);
         seq_item_port.item_done ();
      end
   endtask

   virtual task drive_item (my_data data_obj);
      // Drive based on bus protocol
   endtask
endclass
```





# other details

드라이버 포트와 시퀸서의 내보내기는 connect_phase() 에서 연결됨

```verilog
virtual function void connect_phase ();
   m_drv0.seq_item_port.connect (m_seqr0.seq_item_export);
endfunction
```






