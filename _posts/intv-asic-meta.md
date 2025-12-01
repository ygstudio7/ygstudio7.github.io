# 12) Metastability & CDC

### 질문 의도

정의/원인 → 영향 → 해결방법

### 1분 답변

- **핵심**: 서로 다른 클록 도메인 샘플링 시 FF 입력이 setup/hold 위반 → **메타** 발생.
- **대책**: 2-FF 동기화(단일비트), Gray-code 포인터+2-FF(다중 비트/카운터), 핸드셰이크(ready/valid), **CDC 툴**로 정적 검증.
- **MTBF**: `MTBF ∝ e^(T_res/τ)` — 해결은 **해결시간 확보(슬로 클록/2-FF)**와 입력 토글률↓.

### 3분 답변

개념/원인

- FF가 setup/hold time 조건을 만족하지 못할때, 출력이 일정시간동안 0/1로 안정되지 못하고, 중간 전압 상태에 머무르는 현상
- 물리적으로는 FF 내부의 cross-coupled inverters (SR latch 같은) 가 안정 상태로 수렴하지 못해서, 전압이 느리게 변하거나 예측 불가 상태가 됨 (0,1 또는 x)

해결 방법

- 단일 비트: 2-stage synchronizer
  - 1st FF에서 meta상태가 되어도, 2번째 FF에서 안정된다.
    - 1st FF: setup/hold violation → unstable → meta state
    - 2nd FF: Setup/hold meet → stable
- 멀티 비트:
  - handshake 또는
  - async FIFO사용 (gray code사용해서 address pointer를 전달)