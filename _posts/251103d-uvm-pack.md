

pack/unpack 메소드는 클래스 내부 변수(fields)를 bit, byte 스트림으로 직렬화하고, 다시 역직렬화할 수 있다. 이는 SPI, I2C, UART같은 통신 프로토콜 검증에 유용

- pack(bit array): 비트 배열로 직렬화

- pack_bytes(byte array): 바이트 배열로 직렬화

- pack_ints(int array): 정수 배열로 직렬화

- unpack(bit array): 비트 배열에서 클래스 필드로 역직렬화

- unpack_bytes(byte array): 바이트 배열에서 클래스 필드로 역직렬화

- unpack_ints(int array): 정수 배열에서 역직렬화



# Using automation macros

* 클래스 정의시 uvm_object_utils_begin(TYPE) ... uvm_object_utils_end 및 uvm_field_* 매크로를 사용해 필드 등록시, pack/unpack이 자동으로 필드들을 선서대로 직렬화/역직렬화에 포함시킴

## pack

```cpp
class Packet extends uvm_objects;
  rand bit [3:0]    m_addr;
  rand bit [3:0]    m_wdata;
  rand bit [3:0]    m_rdata;
  rand bit          m_wr;

  `uvm_object_utils_begin(Packet)
    `uvm_field_int(m_addr,     UVM_DEFAULT)
    `uvm_field_int(m_wdata,    UVM_DEFAULT)
    `uvm_field_int(m_rdata,    UVM_DEFAULT)
    `uvm_field_int(m_wr,       UVM_DEFAULT)
  `uvm_object_utils_end

  function new(string name = "Packet);
    super.new(name);
  endfunction
endclass
```



```cpp
class pack_test extends uvm_test;
  `uvm_component_utils(pack_test)
  function new(string name = "pack_test", uvm_component parent=null);
    super.new(name, parent);
  endfunction

  // Declare some arrays to store packed data 
  bit             m_bits[];
  byte unsigned   m_bytes[];
  int unsigned    m_ints[];

  virtual function void build_phase(uvm_phase phase);
    // First create an object of class "Packet"
    Packet m_pkt = Packet::type_id::create("Packet");
    m_pkt.randomize();
    m_pkt.print();

    // Now, call pack functions
    m_pkt.pack(m_bits);
    m_pkt.pack_bytes(m_bytes);
    m_pkt.pack_int(m_ints);

    // Print the array contents
    `uvm_info(get_type_name(), $sformat("m_bits = %p", m_bits), UVM_LOW)
    `uvm_info(get_type_name(), $sformat("m_bytes = %p", m_bytes), UVM_LOW)
    `uvm_info(get_type_name(), $sformat("m_ints = %p", m_ints), UVM_LOW)
  endfunction
endclass
```



```cpp
module tb;
	initial begin
		run_test("pack_test");
	end
endmodule
```



```
UVM_INFO @ 0: reporter [RNTST] Running test pack_test...
--------------------------------
Name       Type      Size  Value
--------------------------------
Packet     Packet    -     @1907
  m_addr   integral  4     'hd  
  m_wdata  integral  4     'h9  
  m_rdata  integral  4     'h6  
  m_wr     integral  1     'h1  
--------------------------------
UVM_INFO testbench.sv(74) @ 0: uvm_test_top [pack_test] m_bits='{'h1, 'h1, 'h0, 'h1, 'h1, 'h0, 'h0, 'h1, 'h0, 'h1, 'h1, 'h0, 'h1}
UVM_INFO testbench.sv(75) @ 0: uvm_test_top [pack_test] m_bytes='{'hd9, 'h68}
UVM_INFO testbench.sv(76) @ 0: uvm_test_top [pack_test] m_ints='{3647471616}
UVM_INFO /playground_lib/uvm-1.2/src/base/uvm_report_server.svh(847) @ 0: reporter [UVM/REPORT/SERVER] 
```





## unpack



```cpp
class unpack_test extends uvm_test;
  `uvm_component_utils(unpack_test)
  function new(string name = "unpack_test", uvm_component parent=null);
    super.new(name, parent);
  endfunction

  // Declare a few arrays to hold packed data for each type of "pack" function
  bit 					m_bits[];
  byte unsigned	m_bytes[];
  int  unsigned	m_ints[];

  // Variables to hold return value from pack function
  int m_val1, m_val2, m_val3;

  virtual function void build_phase(uvm_phase phase);
    Packet m_pkt = Packet::type_id::create("Packet");
    Packet m_pkt2 = Packet::type_id::create("Packet");

    `uvm_info(get_type_name(), $sformatf("Start pack"), UVM_LOW)
    // Randomize the first object, print and pack into bit array, then display
    m_pkt.randomize();
    m_pkt.print();
    m_pkt.pack(m_bits);
    `uvm_info(get_type_name(), $sformatf("packed m_bits=%p", m_bits), UVM_LOW)

    // Randomize the first object, print and pack into byte array, then display
    m_pkt.randomize();
    m_pkt.print();
    m_pkt.pack_bytes(m_bytes);
    `uvm_info(get_type_name(), $sformatf("packed m_bytes=%p", m_bytes), UVM_LOW)

    // Randomize the first object, print and pack into int array, then display
    m_pkt.randomize();
    m_pkt.print();
    m_pkt.pack_ints(m_ints);
    `uvm_info(get_type_name(), $sformatf("packed m_ints=%p", m_ints), UVM_LOW)

    `uvm_info(get_type_name(), $sformatf("Start unpack"), UVM_LOW)
    // Now unpack the packed bit array into the second object, and display
    m_val1 = m_pkt2.unpack(m_bits);
    `uvm_info(get_type_name(), $sformatf("unpacked m_val1=0x%0h", m_val1), UVM_LOW)
    m_pkt2.print();

    // Now unpack the packed byte array into the second object, and display
    m_val2 = m_pkt2.unpack_bytes(m_bytes);
    `uvm_info(get_type_name(), $sformatf("unpacked m_val2=0x%0h", m_val2), UVM_LOW)
    m_pkt2.print();

    // Now unpack the packed int array into the second object, and display
    m_val3 = m_pkt2.unpack_ints(m_ints);
    `uvm_info(get_type_name(), $sformatf("unpacked m_val3=0x%0h", m_val3), UVM_LOW)
    m_pkt2.print();
  endfunction
endclass

module tb;
	initial begin
		run_test("unpack_test");
	end
endmodule
```





# Using do_pack/do_unpack






