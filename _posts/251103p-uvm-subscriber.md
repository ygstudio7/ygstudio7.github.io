# uvm subscriber

구독자는 어떤 분석 포트 (analysis port)에 연결돤 방송자 *(broadcaster)를 듣는 리스너 (listener)이다

분석포트에 아이템이 들어올때마다 해당 객체를 수신한다



uvm_component 자체는 내장된 analysis port가 없지만,

uvm_subscriber는 분석 포트 구현체를 미리 포함하고 있는 확장 버전이다.







# class definition

```verilog
virtual class uvm_subscriber #(type T=int) extends uvm_component;
	typedef uvm_subscriber #(T) this_type;

	uvm_analysis_imp #(T, this_type) analysis_export;

	function new (string name, uvm_component parent);
		super.new (name, parent);
		analysis_export = new ("analysis_imp", this);
	endfunction

	pure virtual function void write (T, t);
endclass
```









# use case

<img src="https://www.chipverify.com/images/uvm/uvm_subscriber.svg" alt="uvm_subscriber_in_agent" style="zoom:150%;" />

위와 같이 uart_agent가 uart_monitor를 통해 트랜잭션 객체를 받아오고, 이 객체를 coverage 콜랙터로 전달하고자 할때 유용하다.

구독자 클래스를 상속하는 것이 반드시 필요한 건 아니지만, 일관된 방식으로 설계시 권장된다.

```verilog
class my_coverage extends uvm_subscriber #(bus_pkt);

	covergroup cg_bus;
		...
	endgroup

	virtual function void write (bus_pkt pkt);
		cg_bus.sample ();
	endfunction
endclass

class my_env extends uvm_env;
	...
	virtual function void connect_phase (uvm_phase phase);
		super.connect_phase (phase);
		my_agent.custom_ap.connect (my_cov.analysis_export);
	endfunction
endclass
```









