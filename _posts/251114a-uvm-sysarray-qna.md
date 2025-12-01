# Q. 새로운 test가 필요하면 test+sequence를 함께 만들어야 하나?



보통 새로운 테스트가 필요하면 test와 sequence를 둘다 추가하나?

###### ChatGPT said:

결론부터 말하면 **반드시 test + sequence 둘 다 추가해야 하는 것은 아니다.**
 상황에 따라 **test만 추가**하거나, **sequence만 추가**, 혹은 **둘 다 추가**할 수도 있다.

아래에 **UVM에서 실제 실무에서 어떻게 하는지** 깔끔하게 정리해줄게.

------

# ✅ 1. 가장 흔한 경우 (실무 70%)

## **✔ “sequence만 추가하는 방식”**

많은 회사/프로젝트는 테스트마다 test 클래스를 만들지 않고
 **하나의 test 안에서 sequence만 바꾸는 방식**을 씀.

예:

```
run_test("systolic_base_test") +UVM_SEQ=systolic_seq_eye
run_test("systolic_base_test") +UVM_SEQ=systolic_seq_all_ones
run_test("systolic_base_test") +UVM_SEQ=systolic_seq_random
```

`base_test`는 환경(env)만 생성하고
 실제 stimulus는 모두 **sequence가 담당**.

**sequence 종류만 바꿔주면 다른 테스트가 됨.**

➡ **빠르고 관리가 쉬움.
 ➡ factory override로 sequence만 바꿔치기 가능.**

------

# ✅ 2. 두 번째로 흔한 경우 (실무 20%)

## **✔ “test만 추가하는 방식”**

sequence는 하나만 두고
 test마다 constraint override 또는 config override로 동작을 바뀜.

예:

```
class systolic_test_eye extends systolic_base_test;
  // same sequence, but A/B pattern을 eye로 override
endclass
```

즉 sequence는 하나인데
 test가 내부에서 값 세팅 방법을 바꿈.

❗ 이건 패턴이 단순한 경우에 사용.

------

# ✅ 3. 덜 흔하지만 필요한 경우 (실무 10%)

## **✔ “test + sequence를 둘 다 만드는 경우”**

이건 “패턴이 아주 다르고”,
 “테스트 플로우 자체가 바뀌는” 경우에 사용.

예:

- reset 중간 삽입 테스트
- backpressure/stall 테스트
- illegal sequence 테스트 (unexpected input)
- corner case sequence

이런 건 seq만 바뀌는 게 아니라
 “테스트의 흐름(run_phase)” 자체가 달라지므로
 test + sequence 둘 다 만드는 게 맞음.

------

# 🎯 정확한 가이드라인

## **🔸 sequence만 추가하면 되는 경우**

- 입력 패턴만 바뀜 (random / eye / all ones / incremental)
- 타이밍은 동일
- test 흐름(run_phase)은 동일
- env / agent / driver 변경 없음

➡ 너의 경우 **random / eye / ones / incremental**은
 ★ sequence만 추가해도 충분함.
 (하지만 test를 분리해도 문제 없음, 선택사항)

------

## **🔸 test만 추가하면 되는 경우**

- 같은 sequence를 쓰되
   constraint만 test에서 바꾸고 싶음
- 또는 test에서 config_db를 값만 바꿔서 다른 동작 유도

예:

```
+ROW_SIZE=16
+ROW_SIZE=32
```

------

## **🔸 test + sequence 둘 다 만들어야 하는 경우**

- reset 중간 삽입
- variable stall / backpressure
- long idle insertions
- error injection
- multi-phase test
- pipeline flush/bubble 테스트

즉 다음에 추가할:

- reset_stress_test
- backpressure_test
- overflow_test
- sparse_test

이런 건 test + sequence 둘 다 필요함.

------

# 📌 지금 우리가 한 방식은?

너의 요청은:

> random test, eye matrix test, all-ones test, incremental test

→ 패턴만 다름
 → driver 동작 동일
 → test 흐름(run_phase) 동일

따라서 **sequence만 만들어도 충분**하지만
 테스트를 외부에서 바로 선택하고 싶다면
 test 클래스를 만들어서 편하게 실행하는 구조도 가능.

➡ 둘 다 정답이지만, sequence만 만들어도 충분한 경우였다.

------

# ✔️ 결론 정리

| 상황                    | 무엇을 추가?              |
| ----------------------- | ------------------------- |
| 패턴만 다름             | **sequence만**            |
| config override만 필요  | **test만**                |
| 테스트 흐름 자체가 다름 | **test + sequence 모두**  |
| 대규모 regression 구조  | base_test + 여러 sequence |