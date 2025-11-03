# Verilog에서 real과 integer 형 변환



Verilog는 SystemVerilog와 달리 형 변환 연산자가 없기 때문에, Verilog에서 `real` 타입과 `integer` 타입 간 변환은 명시적 할당을 통해 수행됩니다. 

---

### 1. **`real` to `integer` 변환**
`real` 값을 `integer`로 변환할 때 소수점 이하 값이 잘립니다(버림).

```verilog
module real_to_integer;
    real real_value;
    integer int_value;

    initial begin
        real_value = 1.23;
        int_value = real_value; // 암시적 변환
        $display("Real to Integer: %f -> %d", real_value, int_value);
    end
endmodule
```

#### 주요 사항:
- `real` 타입을 `integer`에 할당하면 암시적으로 변환됩니다.
- 소수점 이하는 자동으로 잘립니다.

---

### 2. **`integer` to `real` 변환**
`integer` 값을 `real`로 변환할 때는 암시적 할당을 사용합니다.

```verilog
module integer_to_real;
    integer int_value;
    real real_value;

    initial begin
        int_value = 42;
        real_value = int_value; // 암시적 변환
        $display("Integer to Real: %d -> %f", int_value, real_value);
    end
endmodule
```

#### 주요 사항:
- `integer`를 `real`에 할당하면 암시적으로 변환됩니다.
- 정수 값이 실수 값으로 변환되며 소수점 아래는 `0.0`으로 채워집니다.

---

### 3. **`real`과 `integer` 변환 예제**
다음은 두 변환을 포함하는 예제입니다.

```verilog
module test_real_integer_conversion;
    real real_value;
    integer int_value;

    initial begin
        // Real to Integer
        real_value = 1.23;
        int_value = real_value;
        $display("Real to Integer: %f -> %d", real_value, int_value);

        // Integer to Real
        int_value = 123;
        real_value = int_value;
        $display("Integer to Real: %d -> %f", int_value, real_value);

        $finish;
    end
endmodule
```

---

### 주의사항
1. **정확도 손실**:
   - `real`에서 `integer`로 변환할 때 소수점 이하 데이터가 손실됩니다.
   - `real`은 부동소수점 형식이므로, 큰 정수 값의 변환 시 정확도가 떨어질 수 있습니다.
2. **암시적 변환**:
   - Verilog는 명시적 형 변환 연산자가 없으므로 암시적 할당을 통해 변환합니다.
   - 이로 인해 변환 과정이 코드에 명확히 드러나지 않을 수 있습니다.
3. **Verilog `real` 사용 제한**:
   - Verilog에서는 `real` 타입을 레지스터처럼 사용하거나 합성 도구로 구현할 수 없으므로, 변환은 주로 시뮬레이션에서만 사용됩니다.



# Systemverilog에서 real 과 integer 형 변환

SystemVerilog에서 `real` 타입과 `integer` 타입 간 변환은 다음과 같은 방식으로 **명시적**으로 수행할 수 있습니다.

------

### 1. **`real` to `integer` 변환**

`real` 값을 `integer`로 변환할 때 소수점 이하 값은 버려집니다(즉, 내림이 아닌 잘림(truncation)이 발생).

```systemverilog
real real_value = 3.14;
integer int_value;

// 변환
int_value = int'(real_value);
$display("Real to Integer: %f -> %d", real_value, int_value);
```

#### 주요 사항:

- `int'()`를 사용하여 명시적으로 형 변환합니다.
- 잘림(truncation) 방식을 사용하여 정수로 변환합니다. (3.14 -> 3)

------

### 2. **`integer` to `real` 변환**

`integer` 값을 `real`로 변환할 때 정수 값은 실수로 변환됩니다.

```systemverilog
integer int_value = 5;
real real_value;

// 변환
real_value = real'(int_value);
$display("Integer to Real: %d -> %f", int_value, real_value);
```

#### 주요 사항:

- `real'()`를 사용하여 명시적으로 형 변환합니다.
- 정수 값이 실수로 변환되며, 소수점 아래는 0으로 표시됩니다.

------

### 예제: `real`과 `integer` 변환

다음은 위의 변환을 포함한 간단한 테스트 예제입니다.

```systemverilog
module test_real_integer_conversion;
    real real_value;
    integer int_value;

    initial begin
        // Real to Integer
        real_value = 1.23;
        int_value = int'(real_value);
        $display("Real to Integer: %f -> %d", real_value, int_value);

        // Integer to Real
        int_value = 123;
        real_value = real'(int_value);
        $display("Integer to Real: %d -> %f", int_value, real_value);

        // End of simulation
        $finish;
    end
endmodule
```

------

### 주의사항

1. 정확도 손실

   :

   - `real`은 IEEE 754 부동소수점 형식을 사용하므로 큰 값을 정수로 변환하거나, 반대로 변환할 때 정확도가 손실될 수 있습니다.

2. 명시적 형 변환 권장

   :

   - 암시적 변환보다는 명시적 변환(`int'()` 또는 `real'()`)을 사용하여 코드의 명확성을 유지하세요.