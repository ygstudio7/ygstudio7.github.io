# Baes classes

![core-class](https://www.chipverify.com/images/uvm/uml_uvm_base_classes.svg)

이 클래스들은 tb 구성하는 component와 transaction을 위한 골격 역할

## uvm_void

* 매우 기본적인 추상 클래스 (데이터 멤버 X, 함수 X)로 최상단 클래스

* 컨테이너가 여러 타입의 객체를 담을 수 있게 하는 역할

## uvm_object

* uvm_void 상속

* 함수: copy, compare, print, record 등

## ubm_report_object

* reporting 인프라를 위한 인터페이스 제공

* warning, error, info 메시지등 처리

## uvm_component

* tb의 주요 components의 기반 클래스

* 게측 탐색, phasing, reporting, recording, factory 등의 interface 포함

## uvm_transaction

* 단순 transaction용

* sequence 기반 transaction은 uvm_sequence_item을 사용

## uvm_root

* 시뮬레이션 시작시, 자동으로 생성되는 최상위 uvm component

* 글로벌 변수로 접근 가능

* phasing, search, global reporting 등 처리

## 활용 가이드

- 이 기본 클래스들을 이해하면 UVM 기반 검증환경의 골격(scaffolding)을 설계할 때 큰 도움이 됩니다.

- 예컨대 트랜잭션 중심 설계(transaction-centred design)를 할 경우, uvm_transaction 또는 uvm_object 계층을 활용하고, 검증 블록(component) 구축 시 uvm_component와 uvm_root의 역할을 이해하면 구조화가 쉬워집니다.

- 또한 리포팅이나 기록(recording) 요구사항이 있는 검증환경에서는 uvm_report_object에서 제공하는 인터페이스를 적절히 활용해야 합니다.

- 팀이나 IP 재사용 측면에서 “이 클래스는 무엇을 상속해야 하는가?”, “이 컴포넌트는 어떤 기본 클래스를 기반으로 설계되어야 하는가?” 같은 질문을 명확히 할 수 있습니다.

# uvm_object

[UVM utility &amp; field macros](https://www.chipverify.com/uvm/uvm-utility-field-macros)

* 대부분의 객체들의 부모 클래스

* 공통 유틸리티 함수들 정의: print, copy, compare, record

### 정의

```cpp
virtual class uvm_object extends uvm_void;

    // Creates a new uvm_object with the given name, empty by default
    function new(string name="");

    // Utility functions
    function void print (uvm_printer printer=null);
     function void copy (uvm_object rhs, uvm_copier copier=null);
      function bit  compare (uvm_object rhs, uvm_comparer comparer=null);
     function void record (uvm_recorder recorder=null);
    ...

    // These two functions have to be redefined by child classes  
      virtual function uvm_object create (string name=""); return null; endfunction
      virtual function string get_type_name (); return ""; endfunction

endclass
```

create(), get_type_name()은 재정의해야 함

```cpp
class my_object extends uvm_object;
    ...

    // Implementation : Create an object of the new class type and return it
    virtual function uvm_object create(string name="my_object");
        my_object obj = new(name);
        return obj;
    endfunction

endclass
```

```cpp
class my_object extends uvm_object;

    // This static method is used to access via scope operator ::
    // without having to create an object of the class
    static function string type_name();
        return "my_object";
    endfunction

    virtual function string get_type_name();
        return type_name;
    endfunction
endclass
```

### Factory Interface

uvm에서는 클래스 타입을 런타임에 생성할 수 있도록 factory 메카니즘 사용

factory 등록을 위해 매크로가 포함됨: uvm_object_utils, uvm_object_utils_begin/end

```cpp
// An object derived from uvm_object by itself does not get
// registered with the UVM factory unless the macro is called
// within the class definition

class my_object extends uvm_object;

    // Register current object type with the factory
    `uvm_object_utils (my_object)

   ...

endclass
```

### Utility functions

- **print**: 객체의 필드(field)를 출력(print)합니다. 깊이 있는(deep) 출력이 가능하며, 사용자 정의 정보는 `do_print` 훅(hook)을 통해 추가할 수 있습니다. [ChipVerify](https://www.chipverify.com/uvm/uvm-object?utm_source=chatgpt.com)

- **copy**: 다른 객체(rhs)로부터 이 객체에게 필드를 복사(copy)합니다. 사용자 정의 복사를 원할 경우 `do_copy` 훅을 오버라이드할 수 있습니다. [ChipVerify](https://www.chipverify.com/uvm/uvm-object?utm_source=chatgpt.com)

- **compare**: 두 객체를 깊이 비교(deep compare)해서 일치하면 1, 아니면 0을 반환합니다. 사용자 정의 비교 로직을 위해 `do_compare` 훅을 제공합니다. [ChipVerify](https://www.chipverify.com/uvm/uvm-object?utm_source=chatgpt.com)

- **record**: 객체 정보를 기록(record)하는 기능을 가집니다. 로그나 리포트 용도로 사용할 수 있습니다. [ChipVerify](https://www.chipverify.com/uvm/uvm-object?utm_source=chatgpt.com)

# UVM Utility / Utility Macro

## Utility macro

* uvm에서는 factory 메커니즘으로 object, component들을 런타임에 생성하고, 재사용 가능하게 하기 위해 class 등록 (registration)이 필요함. 이를 위해 utility macro를 사용함

### Object utility

* uvm_object_utils 매크로는 uvm_object 나 uvm_transation을 사용한 사용자 정의 클래스에 사용

```cpp
class ABC extends uvm_object;
  `uvm_object_utils(ABC)
  function new(string name="ABC");
    super.new(name);
  endfunction
endclass
```

### Component utility

* uvm_component를 상속한 클래스에는  uvm_component_utils 매크로 사용

```cpp
class DEF extends uvm_component;
  `uvm_component_utils(DEF)
  function new(string name="DEF", uvm_component parent=null);
    super.new(name, parent);
  endfunction
endclass
```

### Macro expansion

내부적으로 다음과 같이 확장된다

```cpp
// Empty uvm_object_utils macro
`define uvm_object_utils(T)
  `uvm_object_utils_begin(T)
  `uvm_object_utils_end

`define uvm_object_utils_begin(T)
   `m_uvm_object_registry_internal(T,T)     // Sub-macro #1
   `m_uvm_object_create_func(T)               // Sub-macro #2
   `m_uvm_get_type_name_func(T)               // Sub-macro #3
   `uvm_field_utils_begin(T)                 // Sub-macro #4

// uvm_object_utils_end simply terminates a function started
// somewhere in the middle
`define uvm_object_utils_end
     end
   endfunction
```

## Field macro

* 사용자 정의 클래스의 멤버 변수에 대해 자동으로, print, copy, compare, pack/unpack, record등의 기능을 제공하기 위해 uvm_field_* 매크로 사용

* uvm_object_utils_begin(TYPE)과 uvm_object_utils_end 블록 내부에 배치
  
  * flag 인자를 사용하여 어떤 자동화 기능을 적용할지 제어: UVM_ALL_ON, UVM_DEFAULT, UVM_NOCOPY, UVM_NOPRINT [내용: website 참고]
  
  * 다양한 데이터형에 대응하는 매크로들: uvm_field_int, uvm_field_enum, uvm_field_sarray_int, uvm_field_queue_object 등

```cpp
class ABC extends uvm_object;
    rand bit [15:0]     m_addr;
    rand bit [15:0]     m_data;

    `uvm_object_utils_begin(ABC)
        `uvm_field_int(m_addr, UVM_DEFAULT)
        `uvm_field_int(m_data, UVM_DEFAULT)
    `uvm_object_utils_end

    function new(string name = "ABC");
        super.new(name);
    endfunction
endclass
```

# UVM Object print

* 객체의 상태를 출력하는 method

* 가상이 아니라서 override 불가 

* 사용자 정의 구현 hook인 do_print()로 출력 방식을 변경 가능

## Example

```cpp
class Object extends uvm_object;
  rand e_bool  m_bool;
  rand bit[3:0] m_mode;
  rand byte m_data[4];
  rand shortint m_queue[$];
  string m_name;
  constraint c_queue { m_queue.size() == 3; }

  function new(string name = "Object");
    super.new(name);
    m_name = name;
  endfunction

  `uvm_object_utils_begin(Object)
    `uvm_field_enum(e_bool,  m_bool,  UVM_DEFAULT)
    `uvm_field_int (m_mode,               UVM_DEFAULT)
    `uvm_field_sarray_int(m_data,         UVM_DEFAULT)
    `uvm_field_queue_int(m_queue,         UVM_DEFAULT)
    `uvm_field_string(m_name,             UVM_DEFAULT)
  `uvm_object_utils_end
endclass
```

 obj.print() 호출시, 위 멤버 변수들이 table형태로 출력됨

```
UVM_INFO @ 0: reporter [RNTST] Running test base_test...
-------------------------------------
Name       Type          Size  Value 
-------------------------------------
obj        Object        -     @1899 
  m_bool   e_bool        32    TRUE  
  m_mode   integral      4     'hd   
  m_data   sa(integral)  4     -     
    [0]    integral      8     'h6c  
    [1]    integral      8     'hf4  
    [2]    integral      8     'he   
    [3]    integral      8     'h58  
  m_queue  da(integral)  3     -     
    [0]    integral      16    'h3cb6
    [1]    integral      16    'h9ae9
    [2]    integral      16    'hd31d
  m_name   string        3     obj   
-------------------------------------
UVM_INFO /playground_lib/uvm-1.2/src/base/uvm_report_server.svh(847) @ 0: reporter [UVM/REPORT/SERVER] 
--- UVM Report Summary ---
```

## Using do_print()

* 매크로 + 추가 정보 출력이 필요한 경우 do_print(uvm_printer printer) 를 override

* 아래와 같이 원하는 변수만 선택, 출력 가능

```cpp
class Object extends uvm_object;
  `uvm_object_utils(Object)  // 오로지 클래스 등록만
  virtual function void do_print(uvm_printer printer);
    super.do_print(printer);
    printer.print_string("m_bool", m_bool.name());
    printer.print_field_int("m_mode", m_mode, $bits(m_mode), UVM_HEX);
    printer.print_string("m_name", m_name);
  endfunction
 UVM Report Summary ---
```

```
UVM_INFO @ 0: reporter [RNTST] Running test base_test...
-------------------------------
Name      Type      Size  Value
-------------------------------
obj       Object    -     @1898
  m_bool  string    4     TRUE 
  m_mode  integral  4     'hd  
  m_name  string    3     obj  
-------------------------------
UVM_INFO /playground_lib/uvm-1.2/src/base/uvm_report_server.svh(847) @ 0: reporter [UVM/REPORT/SERVER] 
--- UVM Report Summary ---
```

## Using sprint

print()와 동일한 형식의 출력물을 문자열 (string) 형태로 변환

```cpp
class base_test extends uvm_test;
  `uvm_component_utils(base_test)
  function new(string name = "base_test", uvm_component parent=null);
    super.new(name, parent);
  endfunction

  function void build_phase(uvm_phase phase);
    Object obj = Object::type_id::create("obj");
    obj.randomize();

    // Instead of calling print() function, let us call "sprint"
    `uvm_info(get_type_name(), $sformatf("Contents: %s", obj.sprint()), UVM_LOW)
  endfunction
endclass

module tb;
    initial begin
        run_test("base_test");
    end
endmodule
```

## Using convert2string

사용자 출력 형식을 자유롭게 정의

```cpp
class Object extends uvm_object;

  rand e_bool                 m_bool;
  rand bit[3:0]             m_mode;
  rand byte                 m_data[4];
  rand shortint             m_queue[$];
  string                     m_name;

  constraint c_queue { m_queue.size() == 3; }

  function new(string name = "Object");
    super.new(name);
    m_name = name;
  endfunction

  // Use "do_print" instead of the automation macro
  `uvm_object_utils(Object)

  virtual function string convert2string();
    string contents = "";
    $sformat(contents, "%s m_name=%s", contents, m_name);
    $sformat(contents, "%s m_bool=%s", contents, m_bool.name());
    $sformat(contents, "%s m_mode=0x%0h", contents, m_mode);
    foreach(m_data[i]) begin
      $sformat(contents, "%s m_data[%0d]=0x%0h", contents, i, m_data[i]);
    end
    return contents;
  endfunction
```

```cpp
class base_test extends uvm_test;
  `uvm_component_utils(base_test)
  function new(string name = "base_test", uvm_component parent=null);
    super.new(name, parent);
  endfunction

  function void build_phase(uvm_phase phase);
    Object obj = Object::type_id::create("obj");
    obj.randomize();
    `uvm_info(get_type_name(), $sformatf("convert2string: %s", obj.convert2string()), UVM_LOW)
  endfunction
endclass

module tb;
    initial begin
        run_test("base_test");
    end
endmodule
```

```
UVM_INFO @ 0: 리포터 [RNTST] 테스트 base_test 실행 중...
UVM_INFO testbench.sv(99) @ 0: uvm_test_top [base_test] convert2string:
m_name=obj m_bool=TRUE m_mode=0xe m_data[0]=0xf4 m_data[1]=0xe m_data[2]=0x58 m_data[3]=0xbd
UVM_INFO /playground_lib/uvm-1.2/src/base/uvm_report_server.svh(847) @ 0: 리포터 [UVM/REPORT/SERVER]
--- UVM 보고서 요약 ---
```

# UVM Object Copy/Clone

* uvm_object에는 copy()메소도로, 한 객체의 필드들을 다른 객체로 복사 가능

* clone()는 객체를 새로 생성한뒤, 필드도 copy한다

* copy는 다른 객체 -> 현재 객체로 복사, vs clone 현재 객체 -> 새 객체 생성후 복사

## Using Autonomous macros

클래스정의시 uvm_object_utils_begin/end나 uvm_field_*을 사용하면 객체의 멤버 변수들이 자동으로 copy/clone됨

```cpp
class Packet extends uvm_object;
  rand bit [15:0] m_adddr;

  // Auto macros
  `uvm_object_utils_begin(Packet)
    `uvm_field_int(m_addr, UVM_DEFAULT)
  `uvm_object_utils_end

  function new(string name = "Packet");
    super.new(name);
  endfunction
endclass

```



```cpp
class Object extends uvm_objects;
  rand shortint   m_queue[$];
  string          m_name;
  rand Packet     m_pkt;

  function new(string name = "Object");
    super.new(name);
    m_name = name;
    m_pkt = Packet::type_id::create("m_pkt);
    m_pkt.randomize();
  endfunction


  `uvm_object_utils_begin(Object)
    `uvm_field_string(m_name,     UVM_DEFAULT)
    `uvm_field_queue_int(m_queue, UVM_DEFAULT)
    `uvm_field_object(m_pkt,      UVM_DEFAULT)
  `uvm_object_utils_end
endclass
```

```cpp
class base_test extends uvm_test;
  `uvm_component_utils(base_test)

  function new(string name = "base_test", uvm_component parent=null);
    super.new(name, parent);    
  endfunction

  function void build_phase(uvm_phase phase)
    Object obj1 = Object::type_id::create("obj1");
    Object obj2 = Object::type_id::create("obj2");
    obj1.randomize();
    obj1.print();
    obj2.randomize();
    obj2.print();

    obj2.copy(obj1);
    `uvm_info("TEST", "After copy", UVM_LOW)
    obj2.print();
  endfunction
endclass
```

```cpp
module tb;
  initial begin
    run_test("base_test");
endmodule
```

위 처럼, copy 호출시 m_addr도 복사 된다

```
ncsim> run
UVM_INFO @ 0: reporter [RNTST] Running test base_test...
--------------------------------------
Name        Type          Size  Value 
--------------------------------------
obj1        Object        -     @1903 
  m_queue   da(integral)  3     -     
    [0]     integral      16    'h9ae9
    [1]     integral      16    'hd31d
    [2]     integral      16    'ha96c
  m_name    string        4     obj1  
  m_pkt     Packet        -     @1906 
    m_addr  integral      16    'h3cb6
--------------------------------------
--------------------------------------
Name        Type          Size  Value 
--------------------------------------
obj2        Object        -     @1908 
  m_queue   da(integral)  3     -     
    [0]     integral      16    'he17f
    [1]     integral      16    'h98e6
    [2]     integral      16    'h5a41
  m_name    string        4     obj2  
  m_pkt     Packet        -     @1910 
    m_addr  integral      16    'h64c1
--------------------------------------
UVM_INFO testbench.sv(60) @ 0: uvm_test_top [TEST] After copy
--------------------------------------
Name        Type          Size  Value 
--------------------------------------
obj2        Object        -     @1908 
  m_queue   da(integral)  3     -     
    [0]     integral      16    'h9ae9
    [1]     integral      16    'hd31d
    [2]     integral      16    'ha96c
  m_name    string        4     obj1  
  m_pkt     Packet        -     @1909 
    m_addr  integral      16    'h3cb6
--------------------------------------
UVM_INFO /playground_lib/uvm-1.2/src/base/uvm_report_server.svh(847) @ 0: reporter [UVM/REPORT/SERVER] 
--- UVM Report Summary ---
```

위에서 "After copy"이후 obj2에는 obj1의 값이 들어가 있다. m_queue, m_name, m_pkt도 obj1의 값들이다.



## Using do_copy

자동 copy()를 사용하지 않고, 필드 복사를 직접 제어하고 싶다면 do_copy(uvm_object rhs) 를 override. copy()는 내부적으로 do_copy()를 실행

```cpp
class Object extends uvm_objects;
  rand shortint   m_queue[$];
  string          m_name;
  rand Packet     m_pkt;

  function new(string name = "Object");
    super.new(name);
    m_name = name;
    m_pkt = Packet::type_id::create("m_pkt);
    m_pkt.randomize();
  endfunction

  `uvm_object_utils(Object)

  virtual function void do_copy(uvm_object rhs);
    Object _obj;
    super.do_copy(rhs);
    $cast(_obj, rhs);
    m_queue = _obj.m_queue;
    m_name = _obj.m_name;
    m_pkt.copy(_obj.m_pkt);
    `uvm_info(get_name(), "In Object::do_copy()")
  endfunction
endclass
```

## Using clone method

 clone()는 객체를 복제하는 것으로, type_id::create()나 new() + copy()조합으로 구현 가능



```cpp
class base_test extends uvm_tests;
  `uvm_component_utils(base_test);
  function new(string name="base_test", uvm_component parent=null);
    super.new(name, parent);
  endfunction

  function void build_phase(uvm_phase phase);
    Object obj2;
    Object obj1 = Object::type_id::create("obj1");
    obj1.randomize();
    `uvm_info("TEST", $sformatf("Obj1.print: %s", obj1.convert2string()), UVM_LOW);

    // Use $cast to clone obj1 into obj2
    $cast(obj2, obj1.clone());
    `uvm_info("TEST", "After clone", UVM_LOW)
    `uvm_info("TEST", $sformatf("Obj2.print: %s", obj2.convert2string()), UVM_LOW);
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
ncsim> run
UVM_INFO @ 0: reporter [RNTST] Running test base_test...
UVM_INFO testbench.sv(105) @ 0: uvm_test_top [TEST] Obj1.print:  m_name=obj1 m_bool=TRUE m_mode=0xe m_data[0]=0xf4 m_data[1]=0xe m_data[2]=0x58 m_data[3]=0xbd
UVM_INFO testbench.sv(20) @ 0: reporter [m_pkt] In Packet::do_copy()
UVM_INFO testbench.sv(86) @ 0: reporter [obj1] In Object::do_copy()
UVM_INFO testbench.sv(108) @ 0: uvm_test_top [TEST] After clone
UVM_INFO testbench.sv(109) @ 0: uvm_test_top [TEST] Obj2.print:  m_name=obj1 m_bool=TRUE m_mode=0xe m_data[0]=0xf4 m_data[1]=0xe m_data[2]=0x58 m_data[3]=0xbd
UVM_INFO /playground_lib/uvm-1.2/src/base/uvm_report_server.svh(847) @ 0: reporter [UVM/REPORT/SERVER]
```


