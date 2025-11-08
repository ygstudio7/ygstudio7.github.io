[ChatGPT](https://chatgpt.com/c/6909a2e8-ec08-832c-959b-c38d2fe6fc08)

[UVM Test [uvm_test]](https://www.chipverify.com/uvm/uvm-test)



# uvm test



# testcase

* testcase: design의 특정 기능(feature) 및 동작 (functionalities)를 검증하기 위해 작성된 패턴

* 검증계획: 검증에 필요한 기능들 나열

* 동일한 tb를 여러 test에서 재사용 가능, 각기 다른 설정으로 테스트 시나리오를 바꾸는 것이 효율적

# class hierarchy

(-) 여러 testcase에 동일한 코드를 작성 X

전체 testbench를 environment라는 컨테이너에 넣고, 각 test에 대해 서로 다른 구성을 가진 동일한 환경 사용



<img src="https://www.chipverify.com/images/uvm/uvm_test.svg" title="" alt="uvm_테스트" width="661">



<img src="https://www.chipverify.com/images/uvm/uml_uvm_test.svg" title="" alt="uvm_test_inheritance" width="273">





# steps to write a uvm test

1. uvm_test에서 상속받은 my_test 클래스 생성하고, factory에 등록(uvm_component_utils())하고, new를 호출
   
   1. 아래에서 build_phase, run_phase등을 구현

```verilog
// Step 1: Declare a new class that derives from "uvm_test"
// my_test is user-given name for this class that has been derived from "uvm_test"
class my_test extends uvm_test;

    // [Recommended] Makes this test more re-usable
    `uvm_component_utils (my_test)

    // This is standard code for all components
    function new (string name = "my_test", uvm_component parent = null);
      super.new (name, parent);
    endfunction

    // Code for rest of the steps come here
endclass
```

2. 다른 환경변수와 검증 요소들을 정의하고 생성
   
   1. my_env, my_cfg를 정의
   
   2. build_phase에서 이들을 생성하고, 
   
   3. uvm_config_db를 통해 설정 객체를 환경내에 등록한다.

```verilog
// Step 2: Declare other testbench components - my_env and my_cfg are assumed to be defined
      my_env   m_top_env;              // Testbench environment that contains other agents, register models, etc
      my_cfg   m_cfg0;                 // Configuration object to tweak the environment for this test

      // Instantiate and build components declared above
      virtual function void build_phase (uvm_phase phase);
         super.build_phase (phase);

         // [Recommended] Instantiate components using "type_id::create()" method instead of new()
         m_top_env  = my_env::type_id::create ("m_top_env", this);
         m_cfg0     = my_cfg::type_id::create ("m_cfg0", this);

         // [Optional] Configure testbench components if required, get virtual interface handles, etc
         set_cfg_params ();

         // [Recommended] Make the cfg object available to all components in environment/agent/etc
         uvm_config_db #(my_cfg) :: set (this, "m_top_env.my_agent", "m_cfg0", m_cfg0);
      endfunction
```

3. 필요하면 UVM topology를 출력

```verilog
// [Recommended] By this phase, the environment is all set up so its good to just print the topology for debug
      virtual function void end_of_elaboration_phase (uvm_phase phase);
         uvm_top.print_topology ();
      endfunction
```

4. virtual sequence 시작
   
   1. create로 sequence 생성
   
   2. start로 시퀀스 시작

```verilog
// Start a virtual sequence or a normal sequence for this particular test
virtual task run_phase (uvm_phase phase);

	// Create and instantiate the sequence
	my_seq m_seq = my_seq::type_id::create ("m_seq");

	// Raise objection - else this test will not consume simulation time*
	phase.raise_objection (this);

	// Start the sequence on a given sequencer
	m_seq.start (m_env.seqr);

	// Drop objection - else this test will not finish
	phase.drop_objection (this);
endtask
```

# how to run a uvm test

 

```verilog
// Specify the testname as an argument to the run_test () task
initial begin
   run_test ("base_test");
end
```

run_test는 아래와 같이 구성됨

```verilog
// This is a global task that gets the UVM root instance and
// starts the test using its name. This task is called in tb_top
task run_test (string test_name="");
  uvm_root top;
  uvm_coreservice_t cs;
  cs = uvm_coreservice_t::get();
  top = cs.get_root();
  top.run_test(test_name);
endtask
```

# how to run any uvm test

* 다른 test 실행시마다, testbench 수정 X -> 다양한 테스트 가능

* +UVM_TESTNAME 으로 test 지정

```verilog
// Pass the DEFAULT test to be run if nothing is provided through command-line
initial begin
   run_test ("base_test");
   // Or you can leave the argument as blank
   // run_test ();
end

// Command-line arguments for an EDA simulator
$> [simulator] -f list +UVM_TESTNAME=base_test
```



# uvm base test example



```verilog
// Step 1: Declare a new class that derives from "uvm_test"
class base_test extends uvm_test;

	  // Step 2: Register this class with UVM Factory
   `uvm_component_utils (base_test)

   // Step 3: Define the "new" function
   function new (string name, uvm_component parent = null);
      super.new (name, parent);
   endfunction

   // Step 4: Declare other testbench components
   my_env   m_top_env;              // Testbench environment
   my_cfg   m_cfg0;                 // Configuration object


   // Step 5: Instantiate and build components declared above
   virtual function void build_phase (uvm_phase phase);
      super.build_phase (phase);

      // [Recommended] Instantiate components using "type_id::create()" method instead of new()
      m_top_env  = my_env::type_id::create ("m_top_env", this);
      m_cfg0     = my_cfg::type_id::create ("m_cfg0", this);

      // [Optional] Configure testbench components if required
      set_cfg_params ();

      // [Optional] Make the cfg object available to all components in environment/agent/etc
      uvm_config_db #(my_cfg) :: set (this, "m_top_env.my_agent", "m_cfg0", m_cfg0);
   endfunction

   // [Optional] Define testbench configuration parameters, if its applicable
   virtual function void set_cfg_params ();
      // Get DUT interface from top module into the cfg object
      if (! uvm_config_db #(virtual dut_if) :: get (this, "", "dut_if", m_cfg0.vif)) begin
         `uvm_error (get_type_name (), "DUT Interface not found !")
      end

      // Assign other parameters to the configuration object that has to be used in testbench
      m_cfg0.m_verbosity    = UVM_HIGH;
      m_cfg0.active         = UVM_ACTIVE;
   endfunction

	  // [Recommended] By this phase, the environment is all set up so its good to just print the topology for debug
   virtual function void end_of_elaboration_phase (uvm_phase phase);
      uvm_top.print_topology ();
   endfunction

   function void start_of_simulation_phase (uvm_phase phase);
      super.start_of_simulation_phase (phase);

      // [Optional] Assign a default sequence to be executed by the sequencer or look at the run_phase ...
      uvm_config_db#(uvm_object_wrapper)::set(this,"m_top_env.my_agent.m_seqr0.main_phase",
                                       "default_sequence", base_sequence::type_id::get());

   endfunction

   // or [Recommended] start a sequence for this particular test
   virtual task run_phase (uvm_phase phase);
   	my_seq m_seq = my_seq::type_id::create ("m_seq");

   	super.run_phase(phase);
   	phase.raise_objection (this);
   	m_seq.start (m_env.seqr);
   	phase.drop_objection (this);
   endtask
endclass
```



# Derivative tests

base test를 작성후, 이를 상속한 파생 테스트를 통해 다른 시퀀스나 설정을 바뀌 실행할 수 있다

-> (+) testcode의 재사용성과 유연성 증가

```verilog
class dv_wr_rd_register_test extends base_test;
  `uvm_component_utils(dv_wr_rd_register_test)
  function new(string name = "dv_wr_rd_register_test", uvm_component parent = null);
    super.new(name, parent);
  endfunction

  virtual task run_phase(uvm_phase phase);
    wr_rd_reg_seq m_wr_rd_reg_seq = wr_rd_reg_seq::type_id::create("m_wr_rd_reg_seq");
    m_wr_rd_reg_seq.start(m_top_env.my_agent.my_sequencer);
  endtask
endclass

```



다음은 config를 바꾼 파생 클래스

```verilog
// Build a derivative test that builds a different configuration
// base_test <- dv_wr_rd_register_test <- dv_cfg1_wr_rd_register_test

class dv_cfg1_wr_rd_register_test extends dv_wr_rd_register_test;
	`uvm_component_utils (dv_cfg1_wr_rd_register_test)

	function new(string name = "dv_cfg1_wr_rd_register_test");
		super.new(name);
	endfunction

	// First calls base_test build_phase which sets m_cfg0.active to ACTIVE
	// and then here it reconfigures it to PASSIVE
	virtual function void build_phase(uvm_phase phase);
		super.build_phase(phase);
		m_cfg0.active = UVM_PASSIVE;
	endfunction
endclass
```
