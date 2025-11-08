* 단일 클래스

* tb에서 모든  uvm components를 포함하는 최상위 컨테이너: uvm_top으로 접근 가능

* 시뮬레이션 시작시, uvm이 자동으로 uvm_root 인스턴스 생성

# Implicit Top-Level

* 부모 지정없이 생성된 컴포넌트는 자동으로 uvm_top의 child가 됨

* uvm_top이 전체 컴포넌트에 대해 build/connect/run 등의 phase 실행 제어



```cpp

class my_env extends uvm_env;
	`uvm_component_utils (my_env)

	// Note that if parent is not explicitly passed during creation, it will be null
	function new(string name = "my_env", uvm_component parent = null);
		super.new(name, parent);
	endfunction

endclass

class my_test extends uvm_test;
	`uvm_component_utils (my_test)

	my_env 		m_env_1;
	my_env 		m_env_2;
	...

	virtual function void build_phase(uvm_phase phase);
		super.build_phase(phase);

		// Parent is set to my_test
		m_env_1 = my_env::type_id::create("m_env_1", this);

		// Parent is null, and hence set to uvm_top
		m_env_2 = my_env::type_id::create("m_env_2");
	endfunction
endclass
```

m_env2도 parent가 없지만, 묵시적으로 uvm_top아래 들어간다 



# Search Functionality

* uvm_top으로부터 이름이나 wildcard(*)로 컴포넌트 검색 가능: find(), find_all()



```cpp
class my_test extends uvm_test;
	`uvm_component_utils (my_test)

	my_env 		m_env;

	virtual function void build_phase(uvm_phase phase);
		super.build_phase(phase);

		m_env = my_env::type_id::create("m_env", this);
	endfunction

	virtual function void end_of_elaboration_phase(uvm_phase phase);
		uvm_component 	comp;
		uvm_component 	comp_q [$];

		super.end_of_elaboration_phase(phase);

		// Gets handle to the driver indicated by the path
		comp = uvm_top.find("m_env.m_agent.m_driver");

		// Gets the first handle to the driver that matches the pattern
		comp = uvm_top.find("*apb_driver*");

		// Gets queue of all handles that match the pattern
		comp_q = uvm_top.find_all("*apb_driver*");

		foreach (comp_q[i]) begin
			`uvm_info (get_type_name(), $sformatf("Found component %s", comp_q[i].get_full_name()), UVM_LOW)
		end

	endfunction
endclass
```



# Report Configuration

* uvm_top을 통해 전체 시뮬레이션의 리포트 메시지를 글로벌하게 설정가능

* 리포트 설정을 변경하고자 하면 e.g. `uvm_top.set_report_verbosity_level_hier(UVM_HIGH);` 등의 메소드를 이용해 전체 로그 수준을 한 번에 조정





# 
