# uvm_component

* uvm_component는 tb에서 사용되는 모든 주요 컴포넌트 (driver, monitor, agent, scoreboard) 의 기본 클래스

* 계층 구조: 트리 형태 구조 탐색을 위해 부모, 자식 관계를 구성

* phasing: 시뮬레이션 단계를 정의해서, build -> connect -> run 과 같이, 각 components들이 이 단계를 따라 동작

* Reporting: 메시지, 경고, 오류등 공용 리포팅 인프라 사용

* factory 메커니즘: 런타임시 클래스 인스턴스를 생성하고 재사용할 수 있도록 팩토리 등록 및 생성 인터페이스 제공
  
   



# class hierarchy

* uvm_component는 uvm_report_object를 상속

* 

![uvm_component_class_hier](https://www.chipverify.com/images/uvm/uml_uvm_component.svg)

* driver, monitor, scoreboard와 괕은 주요 tb 구서용소들은 uvm_component의 자식으로, 표준 phasing 및 reporting 방식을 사용

![uvm_component_uvc](https://www.chipverify.com/images/uvm/uml_uvm_component_uvc.svg)



* 관련 함수들

```cpp
// Get handle to parent uvm component that instantiated current component
extern virtual function uvm_component get_parent ();

// Get full hierarchical name of the current component
extern virtual function string get_full_name ();

// Get queue of all children instantiated under current component, not nested
 	extern function void get_children(ref uvm_component children[$]);

// Get handle to a child by the given name
 	extern function uvm_component get_child (string name);

// Used in an iterable to get the next child component handle
  	extern function int get_next_child (ref string name);

// Get the first child handle under current component
  	extern function int get_first_child (ref string name);

// Get total number of child components, not nested
  	extern function int get_num_children ();

// Returns 1 if there is atleast one component instantiated
  	extern function int has_child (string name);

// Get nested child component handle by hierarchical path
  	extern function uvm_component lookup (string name);

// Depth of the current component from uvm_top=0, uvm_test=1, etc
  	extern function int unsigned get_depth();
```



* 아래 예를 보면,

![uvm_component_hier_example](https://www.chipverify.com/images/uvm/uvm_component_hier_example.svg)



```cpp
class apb_monitor extends uvm_component;

class apb_agent extends uvm_component;
  apb_monitor     m_apb_monitor;
endclass

class child_comp extends uvm_component;

class top_comp extends uvm_component;

  // Child component 
  child_comp        m_child_comp_sa[5];
  apb_agent        m_apb_agent;


  virtual function void end_of_eaboration_phase(uvm_phase phase);
    // Local variables to hold temp results
    uvm_component        l_comp_h;
    uvm_component        l_comp_q[$];

    // Get parent and print it
    l_comp_h = get_parent();
    `uvm_info("tag", $sformatf("get_parent=%s", l_comp_h.get_name()), UVM_LOW)

    // Get all children and print them
    get_children(l_comp_q);
    foreach(l_comp_q[i)
        `uvm_info("tag", $sformatf("child %0d = %s", i, l_comp_q[i].get_name()), UVM_LOW);

    // Get a specific child component and print name
    l_comp_h = get_child("m_child_comp_2");
    `uvm_info("tag", $sformatf("Found %s", l_comp_h.get_name()), UVM_LOW);

    // Other function calls
    `uvm_info("tag", $sformatf("number of children = %0d", get_num_children()), UVM_LOW);
    `uvm_info("tag", $sformatf("has_child('abc') = %0d", has_child("abc")), UVM_LOW);
    `uvm_info("tag", $sformatf("has_child('m_apb_agent') = %0d", has_child("m_apb_agent")), UVM_LOW);
    `uvm_info("tag", $sformatf("get_depth = %0d", get_depth()), UVM_LOW);

		// Lookup failure will result in a warning
    	l_comp_h = lookup("m_apb_monitor");
    	if (l_comp_h)
      		`uvm_info ("tag", $sformatf("Found %s", l_comp_h.get_full_name()), UVM_LOW)

		// Lookup requires full hierarchical path to the component
      	l_comp_h = lookup("m_apb_agent.m_apb_monitor");
    	if (l_comp_h)
      		`uvm_info ("tag", $sformatf("Found %s", l_comp_h.get_full_name()), UVM_LOW)

  endfunction
endclass

```



```
UVM_INFO @ 0: reporter [RNTST] Running test my_test...
UVM_INFO testbench.sv(67) @ 0: uvm_test_top.m_top_comp [tag] get_parent=uvm_test_top
UVM_INFO testbench.sv(71) @ 0: uvm_test_top.m_top_comp [tag] child_0 = m_apb_agent
UVM_INFO testbench.sv(71) @ 0: uvm_test_top.m_top_comp [tag] child_1 = m_child_comp_0
UVM_INFO testbench.sv(71) @ 0: uvm_test_top.m_top_comp [tag] child_2 = m_child_comp_1
UVM_INFO testbench.sv(71) @ 0: uvm_test_top.m_top_comp [tag] child_3 = m_child_comp_2
UVM_INFO testbench.sv(71) @ 0: uvm_test_top.m_top_comp [tag] child_4 = m_child_comp_3
UVM_INFO testbench.sv(71) @ 0: uvm_test_top.m_top_comp [tag] child_5 = m_child_comp_4
UVM_INFO testbench.sv(75) @ 0: uvm_test_top.m_top_comp [tag] Found m_child_comp_2
UVM_INFO testbench.sv(77) @ 0: uvm_test_top.m_top_comp [tag] number of children = 6
UVM_INFO testbench.sv(79) @ 0: uvm_test_top.m_top_comp [tag] has_child('abc') = 0
UVM_INFO testbench.sv(80) @ 0: uvm_test_top.m_top_comp [tag] has_child('m_apb_agent') = 1
UVM_INFO testbench.sv(81) @ 0: uvm_test_top.m_top_comp [tag] get_depth = 2
UVM_WARNING /xcelium20.09/tools//methodology/UVM/CDNS-1.2/sv/src/base/uvm_component.svh(2017) @ 0: uvm_test_top.m_top_comp [Lookup Error] Cannot find child m_apb_monitor
```





# Phasing Methods

* 대표적인 phase methods들

* build_phase(uvm_phase phase): 인스턴스 생성하고, 내부 구조 구성

* connect_phase (uvm_phase phase): component들 연결 

* run_phase (uvm_phase phase): 실제 시뮬레이션 시간 (time)을 소비하는 동작을 수행



```cpp
class apb_agent extends uvm_component;
  apb_monitor    m_monitor;
  apb_driver     m_driver;
  apb_sequencer  m_seqr;

  // All children of this component are instantiated in build_phase
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);

    // obj_type::type_id::create is a way to get an object of the type "obj_type"
    // from factory
    m_monitor = apb_monitor::type_id::create("m_monitor", this);
    m_driver  = apb_driver::type_id::create("m_driver", this);
    m_seqr    = apb_sequencer::type_id::create("m_seqr", this);
  endfunction

  // Child components can be connected together in connect_phase
  virtual function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    
    // Connect suqeuencer with driver
    m_driver.seq_item_port.connect(m_seqr.seq_item_export);
  endfunction
endclass
```





# Factory Methods

* components는 여기서만 사용하도록 설계된 `uvm_component_utils` 매크로를 사용하여 factory에 등록됨

```cpp
class my_comp extends uvm_component;

	// Register with factory
	`uvm_component_utils (my_comp)

	function new(string name = "my_comp", uvm_component parent = null);
		super.new(name, parent);
	endfunction

	...

endclass
```



```cpp
// For example, "set_type_override_by_type" is actually a function defined in the class uvm_factory
// A wrapper function with the same name is defined in uvm_component class and arguments are passed
// to the uvm_factory function call

function void uvm_component::set_type_override_by_type (uvm_object_wrapper original_type,
                                                        uvm_object_wrapper override_type,
                                                        bit    replace=1);

	// Get handle to the factory instance in UVM environment
	uvm_coreservice_t cs = uvm_coreservice_t::get();
  	uvm_factory factory = cs.get_factory();

	// Pass the same arguments to the factory function call
   	factory.set_type_override_by_type(original_type, override_type, replace);
endfunction

// Usage : "m_comp" is a uvm_component class handle
m_comp.set_type_override_by_type(apb_driver::get_type(), apb_driver_2::get_type());
```


