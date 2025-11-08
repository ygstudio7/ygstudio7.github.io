참고: [UVM Object Compare](https://www.chipverify.com/uvm/uvm-object-compare)

compare는 자신과 인자로 주어진 필드들을 깊이 비교하고, 모두 동일한 경우 1, 하나라도 차이가 있는 경우 0를 반환

compare는 override X, 사용자 비교를 위해서는 do_compare를 override한다

비교시, uvm_comparer 객체가 사용될 수 있다

## Using automation macros

uvm_fild_* 매크로는, 비교 대상 필드를 자동 등록한다. 따라서, compare()호출시 해당필드들이 자동 비교된다.

```cpp
class base_test extends uvm_test;
  `uvm_component_utils(base_test)
  function new(string name="base_test", uvm_omponent parent=null);
    super.new(name, parent);
  endfunction

  function void build_phase(uvm_phase phase);
    Object obj1 = Object::type_id::create("obj1");
    Object obj2 = Object::type_id::create("obj2");
    obj1.randomize();
    obj1.print();
    obj2.randomize();
    obj2.print();

    _compare(obj1, obj2);

    `uvm_info("TEST", "Copy m_name", UVM_LOW)
    obj2.m_name = obj1.m_name;
    `uvm_info("TEST", $sformatf("Obj2.print: %s", obj2.convert2string()), UVM_LOW)
    _compare(obj1, obj2);

    `uvm_info("TEST", "Copy m_pkt.m_addr", UVM_LOW)
    obj2.m_pkt.m_addr = obj1.m_pkt.m_addr;
    `uvm_info("TEST", $sformatf("Obj2.print: %s", obj2.convert2string()), UVM_LOW)
    _compare(obj1, obj2);

  endfunction

  function void _compare(Object obj1, obj2);
    if(obj2.compare(obj1))
      `uvm_info("TEST", "obj1 and obj2 are same", UVM_LOW)
    else
      `uvm_info("TEST", "obj1 and obj2 are different", UVM_LOW)
  endfunction
endclass
```

```cpp
module tb;
    initial begin
        run_test("base_test");
    end
endmodule
```

```
UVM_INFO @ 0: reporter [RNTST] Running test base_test...
----------------------------------
Name        Type      Size  Value 
----------------------------------
obj1        Object    -     @1903 
  m_bool    e_bool    32    TRUE  
  m_mode    integral  4     'h9   
  m_name    string    4     obj1  
  m_pkt     Packet    -     @1905 
    m_addr  integral  16    'h3cb6
----------------------------------
----------------------------------
Name        Type      Size  Value 
----------------------------------
obj2        Object    -     @1907 
  m_bool    e_bool    32    FALSE 
  m_mode    integral  4     'hf   
  m_name    string    4     obj2  
  m_pkt     Packet    -     @1908 
    m_addr  integral  16    'h64c1
----------------------------------
UVM_INFO /playground_lib/uvm-1.2/src/base/uvm_comparer.svh(351) @ 0: reporter [MISCMP] Miscompare for obj2.m_bool: lhs = FALSE : rhs = TRUE
UVM_INFO /playground_lib/uvm-1.2/src/base/uvm_comparer.svh(382) @ 0: reporter [MISCMP] 1 Miscompare(s) for object obj1@1903 vs. obj2@1907
UVM_INFO testbench.sv(71) @ 0: uvm_test_top [TEST] obj1 and obj2 are different
UVM_INFO testbench.sv(73) @ 0: uvm_test_top [TEST] Copy m_bool
UVM_INFO /playground_lib/uvm-1.2/src/base/uvm_comparer.svh(351) @ 0: reporter [MISCMP] Miscompare for obj2.m_mode: lhs = 'hf : rhs = 'h9
UVM_INFO /playground_lib/uvm-1.2/src/base/uvm_comparer.svh(382) @ 0: reporter [MISCMP] 1 Miscompare(s) for object obj1@1903 vs. obj2@1907
UVM_INFO testbench.sv(78) @ 0: uvm_test_top [TEST] obj1 and obj2 are different
UVM_INFO testbench.sv(80) @ 0: uvm_test_top [TEST] Copy m_mode
UVM_INFO /playground_lib/uvm-1.2/src/base/uvm_comparer.svh(351) @ 0: reporter [MISCMP] Miscompare for obj2.m_name: lhs = "obj2" : rhs = "obj1"
UVM_INFO /playground_lib/uvm-1.2/src/base/uvm_comparer.svh(382) @ 0: reporter [MISCMP] 1 Miscompare(s) for object obj1@1903 vs. obj2@1907
UVM_INFO testbench.sv(85) @ 0: uvm_test_top [TEST] obj1 and obj2 are different
UVM_INFO testbench.sv(87) @ 0: uvm_test_top [TEST] Copy m_name
UVM_INFO /playground_lib/uvm-1.2/src/base/uvm_comparer.svh(351) @ 0: reporter [MISCMP] Miscompare for obj2.m_pkt.m_addr: lhs = 'h64c1 : rhs = 'h3cb6
UVM_INFO /playground_lib/uvm-1.2/src/base/uvm_comparer.svh(382) @ 0: reporter [MISCMP] 1 Miscompare(s) for object obj1@1903 vs. obj2@1907
UVM_INFO testbench.sv(92) @ 0: uvm_test_top [TEST] obj1 and obj2 are different
UVM_INFO testbench.sv(94) @ 0: uvm_test_top [TEST] Copy m_pkt.m_addr
UVM_INFO testbench.sv(97) @ 0: uvm_test_top [TEST] obj1 and obj2 are same
UVM_INFO /playground_lib/uvm-1.2/src/base/uvm_report_server.svh(847) @ 0: reporter [UVM/REPORT/SERVER] 
```

## Using do_compare

`do_compare()` 훅을 오버라이드하면, 비교 전에 특별한 전처리나 비교 후 처리를 추가할 수 있어, 커스텀 비교 로직이 필요한 객체에 유리
