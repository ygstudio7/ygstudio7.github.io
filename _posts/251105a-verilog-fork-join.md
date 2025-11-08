[ChatGPT](https://chatgpt.com/c/690c4389-2150-8332-bb42-a4984236ac63)

# fork... join/join_any/join_none

* verilog에서 fork, join 구문은 프로세스 제어문중 하나

* 병렬 실행을 위해 사용; 여러개 문장을 동시에 실행시키고 싶을때 사용

# 기본 구조

```verilog
fork
  statement1;
  statement2;
  statement3;
join
```

fork...join 사이에 있는 모든 문장을 동시에 실행

# Example

```verilog
initial begin
  $display("Start time = %0t", $time);

  fork
    begin
      #5 $display("Task 1 done at %0t", $time);
    end

    begin
      #10 $display("Task 2 done at %0t", $time);
    end

    begin
      #7 $display("Task 3 done at %0t", $time);
    end
  join

  $display("All tasks finished at %0t", %time);
end
```

```arduino
Start time = 0
Task 1 done at 5
Task 3 done at 7
Task 2 done at 10
All tasks finished at 10
```

# 차이

| 형태                   | 설명                               |
| -------------------- | -------------------------------- |
| `fork ... join`      | 모든 병렬 문장이 완료될 때까지 기다림 (기본형)      |
| `fork ... join_any`  | **하나라도 완료되면** 바로 다음으로 진행         |
| `fork ... join_none` | **기다리지 않고** 즉시 다음 문장 실행 (비동기 실행) |

# Example

#### 1️⃣ `join_any`

```verilog
initial begin
  fork
    #5 $display("A done");
    #10 $display("B done");
  join_any
  $display("→ Proceed after any finishes");
end
```

출력: 하나라도 완료되면; "A done"이 나오면 다음 실행

```bash
A done
→ Proceed after any finishes
B done
```

#### 2️⃣ `join_none`

```verilog
initial begin
  fork
    #5 $display("A done");
    #10 $display("B done");
  join_none
  $display("→ Proceed immediately without waiting");
end
```

출력: 병렬 실행

```bash
→ Proceed immediately without waiting
A done
B done
```

---

## 🔹 요약

| 키워드         | 동작 방식              | 기다림 여부    |
| ----------- | ------------------ | --------- |
| `join`      | 모든 프로세스 완료 후 다음 진행 | ✅ 기다림     |
| `join_any`  | 하나라도 끝나면 다음 진행     | ⚡ 일부만 기다림 |
| `join_none` | 즉시 다음으로 진행         | ❌ 기다리지    |

# Tip

* 다음과 같이 구분해서 외우면 편하다
  
  * join: 모두가 join 했을때 --> 모두가 완료되었을 때 다음 진행
  
  * join_any: 하나라도 join헀을때 -> 하나 완료되면 다음 진행
  
  * join_none: 아무도 join않했을때 -> 바로 실행
