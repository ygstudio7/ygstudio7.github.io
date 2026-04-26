
SV에서 [$]는 dynamic array or queue이다. 
그리고, 일반 배열 인덱싱으로 바로 접근가능하다

```verilog
shortreal data[$];   // queue 선언
shortreal x;

// push_back으로 추가
data.push_back(1.1); // (5)
data.push_back(2.2);
data.push_back(3.3);

// 전체 개수
$display("size = %0d", data.size()); // (10)

// 개별 접근
$display("first  = %f", data[0]);
$display("second = %f", data[1]);
$display("third  = %f", data[2]); // (15)

// 마지막 요소 접근
$display("last   = %f", data[$]);   // '$'는 마지막 인덱스 의미

// 전체 반복 // (20)
foreach (data[i]) begin
  $display("data[%0d] = %f", i, data[i]);
end
```
(1) 에서 shortreal queue 선언
(5) push_back으로 뒤에 추가
(10) 개수 얻기
(15) 개별 접근방법
(18) data[$] 는 마지막 요소
(21) foreach로 data[i]를 쉽게 접근. 여기서 index i도 implicit으로 자동으로 정의되는 반복 인덱스 변수임



```
size = 3
first  = 1.100000
second = 2.200000
third  = 3.300000
last   = 3.300000
data[0] = 1.100000
data[1] = 2.200000
data[2] = 3.300000
```

## 🔹 자주 쓰는 메서드

| 메서드             | 설명          | 예시                        |
| --------------- | ----------- | ------------------------- |
| `push_back(x)`  | 맨 뒤에 추가     | `data.push_back(val);`    |
| `pop_back()`    | 맨 뒤 제거 및 반환 | `val = data.pop_back();`  |
| `push_front(x)` | 맨 앞에 추가     | `data.push_front(val);`   |
| `pop_front()`   | 맨 앞 제거 및 반환 | `val = data.pop_front();` |
| `delete()`      | 전체 비움       | `data.delete();`          |
| `size()`        | 요소 개수 반환    | `n = data.size();`        |
| `[$]`           | 마지막 요소 인덱스  | `data[$]`                 |
| `[0]`           | 첫 번째 요소     | `data[0]`                 |

# 예 1: 간단 출력

```verilog
shortreal data[$] = '{1.0, 2.0, 3.0};

foreach (data[i]) begin
  $display("data[%0d] = %f", i, data[i]);
end

// 아래는 에러! (foreach 블록 밖에서는 i가 정의되지 않음)
// $display("last i = %0d", i);

```

출력
```
data[0] = 1.000000
data[1] = 2.000000
data[2] = 3.000000
```

# 예 2: 2차원 배열
```verilog
int arr[2][3] = '{'{1,2,3}, '{4,5,6}};

foreach (arr[i,j]) begin
  $display("arr[%0d][%0d] = %0d", i, j, arr[i][j]);
end

```
(3) 여기서 자동 선언 인덱스는 `,` 로 구분한다. (arr [i,j])
실제로는 arr[i][j]로 사용된다.

✅ **정리**

| 형태                            | i 선언 필요? | 범위            | 특징       |
| ----------------------------- | -------- | ------------- | -------- |
| `foreach (data[i])`           | ❌ 자동 정의됨 | foreach 블록 내부 | 간결하고 안전  |
| `for (int i = 0; i < N; i++)` | ✅ 필요     | 선언된 스코프       | 유연하지만 장황 |

