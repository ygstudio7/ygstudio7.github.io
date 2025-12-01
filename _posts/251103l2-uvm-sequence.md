# UVM sequence

* uvm sequence는 여러개의 data item을 조합하여 다양한 자극 scenario를 만드는 클래스
* sequencer에 의해 실행되고, 그 뒤에 driver로 데이터가 전달된다
* sequence는 stimulus generation의 역할 수행

<img src="https://www.chipverify.com/images/uvm/uvm_sequence.svg" alt="sequences on a sequencer" style="zoom:150%;" />

# Class Hierarchy

<img src="https://www.chipverify.com/images/uvm/uml_uvm_sequence.svg" alt="uml_uvm_sequence_class_hier" style="zoom:150%;" />







# Steps to create a UVM sequence

### 1. uvm_sequence를 상속한 클래스 생성후, factory와 `new`를 사용해서 등록

```verilog
// my_sequence is user-given name for this class that has been derived from "uvm_sequence"
class my_sequence extends uvm_sequence;

	// [Recommended] Makes this sequence reusable. Note that we are calling
	// `uvm_object_utils instead of `uvm_component_utils because sequence is a uvm_transaction object
	`uvm_object_utils (my_sequence)

	// This is standard code for all components
    function new (string name = "my_sequence");
    	super.new (name);
    endfunction
endclass
```





### 2. sequence를 실행하는 sequencer를 설정

: 여기서, sequencer가 sequence를 실행하도록 연결시킨다

```verilog
// [Optional] my_sequencer is a pre-defined custom sequencer before this sequence definition
`uvm_declare_p_sequencer (my_sequencer)
```





### 3. body()함수를 정의

```verilog
// [Recommended] Make this task "virtual" so that child classes can override this task definition
virtual task body ();
	// Stimulus for this sequence comes here
endtask
```







# UVM Sequence Example

```verilog
class my_sequence extends uvm_sequence;
	`uvm_object_utils (my_sequence)

	function new (string name = "my_sequence");
		super.new (name);
	endfunction

	// Called before the body() task
	task pre_body ();
		...
	endtask

	task body ();
		my_data pkt;
		`uvm_do (pkt);
	endtask

	// Called after the body() task
	task post_body();
		...
	endtask
endclass
```



uvm_do: data packet을 생성해서 seqr로 보낼때 사용된다.













