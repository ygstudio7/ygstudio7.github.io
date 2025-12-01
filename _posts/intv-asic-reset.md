# 1) Reset (Async vs Sync)

- 정의

  :

  - *Async reset*: 클록과 무관하게 즉시 리셋. 해제(de-assert)는 클록 엣지와 비동기.
  - *Sync reset*: 클록 엣지에서만 리셋이 동작/해제.

- 언제/왜

  :

  - Async

    :

    - (+) power up후, 클록이 안정 전에도 초기화가 필요할 때.

  - Sync

    :

    - (+) reset release에 대한 recovery/removal등 복잡한 타이밍 분석 X → STA툴에서 data line처럼 Timing분석이 단순,
    - (+)  리셋 해제 시점이 클록에 정합되어 글리치/메타 위험 ↓.
    - (+) 저전력 블록/게이트드 클록에서는 sync reset이 유리.

- 주의점

  :

  - Async
    - (-) Release시, **동기화**(2-FF sync)하거나 *reset release* 타이밍 제약(Recovery/Removal) 관리.
  - Sync

- **요약 한 줄**:  `Assert는 async여도 해제는 반드시 클록에 동기화가 핵심.`

## Q1) async reset을 왜 sync release해야 하나?

3가지 이유가 있다.

1. metastability 위험 up:
   1. (+) assert시, clock과 관계없이 동작
   2. (-) deassert시, release 시점이 clock edge 근처이면, 내부 latch가 setup/hold violation이 결려 `출력 Q가 meta상태`로 될 수 있다.
2. sync between domains 필요
   1. (-) 하나의 reset이 수많은 FF에 연결될때, 배션 지연으로 인해 각 FF가 서로 다른 clock 주기에 reset이 해제될 수 있다. 그럼, `FSM이나 register의 동작이 불일치 상태`로 시작되어 문제가 된다.
3. Timing 관리 복잡
   1. (-) async release는 EDA툴에서 recovery/removal 등 북잡하게 분석해야 한다.
   2. (+) sync release는 `setup/hold만 지키면 되는 일반 데이터 경로로 취급`할 수 있다.

https://www.youtube.com/watch?v=3A5Yy-UXi-4

`TODO`: Metastability일때, 실제 중간 전압에 머무를 수 있나?

그래서 0도 1도 아닌 상태가 될 수 있는건가?

아니면, 실제로는 0 아니면 1로 되지만, 그 값이 random이라는 말인가?





# 참고

참고

http://www.sunburst-design.com/papers/CummingsSNUG2003Boston_Resets.pdf