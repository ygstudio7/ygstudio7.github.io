# SV: logic vs bit



참고: https://www.chipverify.com/systemverilog/systemverilog-data-types-logic-bit#google_vignette

| logic | 4 states | 0,1,x,z | default value: X//       |
| ----- | -------- | ------- | ------------------------ |
| bit   | 2 states | 0,1     | (+) simulation faster    |
|       |          |         | z,x값을 넣으면 0이 된다. |

### 

| reg   | only in procedural blocks              | always, initial         |
| ----- | -------------------------------------- | ----------------------- |
| wire  | only in continuous assignment          | assign                  |
| logic | both in procedural blocks & assignment | always, initial, assign |

(Q) 언제 assignment가 유효하나? 언제 en값이 바뀌나? `#1이후? why?`

```jsx
module tb;
  logic [3:0] my_data; // 4 bit
  logic       en;
  
  initial begin
    $display ("my_data=0x%0h, en=%0b", my_data, en); // default value of logic type: X
    my_data = 4'hb;                                  // logic can be driven in initial block
    $display ("my_data=0x%0h, en=%0b", my_data, en); 
    #1;
    $display ("my_data=0x%0h, en=%0b", my_data, en); 
  end
  
  assign en = my_data[0]; // logic can be driven via assign statement
endmodule
```

simulation log

```jsx
my_data=0xx en=x
my_data=0xb en=x
my_data=0xb en=1 // last assignment로 driven됨
```

(`Q`) logic을 bit에 assign하면 어떻게 되나?

```jsx
module tb;
  bit var_a;
  bit [3:0] var_b;
  
  initial begin
    $display("var_a=%0b var_b=0x%0h",var_a, var_b);
    
    var_a = 1;
    var_b = 4'hf;
    $display("var_a=%0b var_b=0x%0h",var_a, var_b);
    
    var_b = 16'h481a;
    $display("var_a=%0b var_b=0x%0h",var_a, var_b);
    
    var_b = 4'b01zx;
    $display("var_a=%0b var_b=%b",var_a, var_b);
  end
endmodule
```

simulation log

```jsx
var_a=0 var_b=0x0
var_a=1 var_b=0xf
var_a=1 var_b=0xa // truncated
var_a=1 var_b=0100 // z,x -> 0 
```





