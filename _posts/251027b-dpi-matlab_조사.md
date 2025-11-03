# Problem

vcs, ncs, modelsim에서 matlab을 코드를 함께 돌려서 검증하는데 사용할 수 있나?

# Research

각 시뮬레이터(vcs, ncs/ncsim(Xcelium), ModelSim/Questa)마다 MATLAB과 연동하는 방식이 조금씩 다릅니다. 

보통 **MathWorks HDL Verifier**(예전 이름: EDA Simulator Link)라는 툴박스를 사용해서 연동합니다.



------

## 1. 기본 개념

- MATLAB/Simulink와 HDL 시뮬레이터(VCS, NCSim, ModelSim/Questa)를 **공동 시뮬레이션(co-simulation)** 할 수 있음.
- 보통 **testbench는 MATLAB/Simulink**, **DUT는 HDL Simulator**에 두고 신호를 주고받으면서 동작 확인.
- 인터페이스는 **shared memory**, **socket**, 또는 **SystemVerilog DPI-C / VHPI / FLI** 등을 통해 연결됨.

------

## 2. 지원 툴

- **Synopsys VCS**: HDL Verifier에서 VCS와 연결 지원. MATLAB에서 cosim 세션을 띄우고 DUT을 VCS에서 시뮬레이션.
- **Cadence NCSim / Xcelium**: 마찬가지로 지원. 연결 스크립트(`nclaunch`, `irun`)와 함께 동작.
- **Mentor ModelSim / QuestaSim**: 가장 흔히 쓰이는 조합. HDL Verifier에 예제가 많음.

------

## 3. 사용하는 방식

1. MATLAB에서 **cosimWizard**를 실행 → HDL 시뮬레이터 선택(VCS, NCSim, ModelSim)
2. DUT 인터페이스 정의 (clk, reset, input, output 포트)
3. MATLAB testbench 작성 후, HDL Simulator를 실행하면 양쪽이 handshake 하면서 신호 교환
   - 예: `hdldaunchsim('vcs')` 또는 `hdldaunchsim('modelsim')` 같은 명령
4. MATLAB 코드 (또는 Simulink 모델)과 RTL 코드가 동시에 실행되며 검증 가능

------

## 4. 실제 검증 시 장점

- **MATLAB에서 golden model** 만들고 RTL 결과와 즉시 비교 가능
- Fixed-point → HDL 검증에 유용 (특히 DSP, 영상처리, 통신 분야)
- testbench를 HDL로 쓰지 않고 MATLAB에서 high-level로 작성 가능

------

👉 정리:
 네, **VCS, NCSim, ModelSim 모두 MATLAB과 직접 cosimulation 가능**합니다. 다만, **MathWorks의 HDL Verifier Toolbox**가 필요하고, 각 시뮬레이터 환경변수/라이선스 설정을 맞춰야 합니다.

------

원하시면, 제가 **예제 코드 흐름** (예: MATLAB에서 VCS와 cosim하는 m-script + Verilog DUT 샘플)도 만들어 드릴까요?

# Question 2

matlab license가 없으면 어떻게 하나?

###### ChatGPT said:

좋은 질문이에요 👍

말씀하신 **MATLAB co-simulation**은 기본적으로 **MATLAB + HDL Verifier Toolbox** 라이선스가 있어야 돌아갑니다.
 만약 MATLAB 라이선스가 없다면, 아래와 같은 대안이 있습니다:

------

## 1. SystemVerilog DPI-C (직접 연동): **

- MATLAB 대신 **C/C++ 모델**을 작성하고, HDL Simulator(VCS, NCSim, ModelSim)와 **DPI-C 인터페이스**로 연결하는 방법입니다.
- MATLAB에서 만든 알고리즘을 **C 코드로 자동 변환**(MATLAB Coder 필요)하거나 직접 C로 다시 작성해서 DPI로 호출.
- 이 경우 MATLAB 실행은 필요 없고, simulator에서 바로 C 모델을 불러와서 비교 가능.
- 장점: MATLAB 라이선스 불필요.
- 단점: MATLAB의 편리한 검증 환경이 사라지고, 변환/디버깅이 필요.

------

## 2. Python 기반 검증 (Cocotb 등)

- MATLAB을 아예 쓰지 않고, **Python + cocotb**를 사용해서 testbench 작성 가능.
- cocotb는 VCS, NCSim, ModelSim/Questa 모두 지원.
- Python에서 **numpy/scipy** 등을 이용해 MATLAB에서 하던 수치 연산/알고리즘 모델링 가능.
- 장점: 무료, 확장성 큼.
- 단점: 기존 MATLAB 코드를 다시 Python으로 옮겨야 함.

------

## 3. SystemC / C++ Reference Model

- MATLAB 대신 **SystemC/C++로 reference model**을 작성하고 HDL과 공동 검증.
- 시뮬레이터에서 VHPI/FLI/DPI 등을 통해 연동 가능.
- 통신/영상/DSP 분야에서는 많이 쓰이는 방식.

------

## 4. MATLAB Online 대체 (제한적)

- 아주 제한된 경우, MATLAB Online (클라우드)이나 무료 Trial 라이선스를 활용할 수 있음.
- 하지만 정식 co-simulation 환경 구축에는 보통 **정식 라이선스**가 필요.

------

✅ 정리하면:

- **MATLAB 라이선스가 없으면 HDL Verifier 기반 co-simulation은 불가능**합니다.
- [v] 대신, **DPI-C / Python (cocotb) / C++ reference model** 같은 방법으로 비슷한 환경을 만들 수 있습니다.

------

원하시면 제가 **"MATLAB 없이 VCS에서 C 모델을 DPI로 연결하는 최소 예제"**를 코드로 짜드릴까요?



# Question 3

matlab에서 c로 컴파일하면, matlab없이 dpi-c 로 연결해서 사용할 수 있나?

# 결론 먼저

- **MATLAB Coder/Embedded Coder**로 생성한 **순수 C/C++ 코드**는 **MATLAB 없이**도 동작합니다.
- 이 코드를 얇은 **DPI-C 래퍼**로 감싸서 **VCS/Xcelium/ModelSim(Questa)** 에 연결하면, MATLAB 없이 HDL과 공동검증할 수 있습니다.
- 주의: **MATLAB Compiler**로 만든 실행물은 **MCR(MATLAB Runtime)** 이 필요합니다. DPI-C 용으로는 **MATLAB Coder(또는 Embedded Coder)** 를 쓰세요.

------

# 전체 흐름(요약)

1. **MATLAB 함수 → C 코드 생성** (MATLAB Coder)
2. 생성된 C 함수 앞뒤로 **DPI-C 어댑터(.c)** 작성
3. **시뮬레이터에서 DPI 컴파일/링크**
4. SystemVerilog에서 `import "DPI-C"`로 불러와서 호출

------

# 최소 예시

### 1) MATLAB 쪽 순수 함수 (예: 단순 FIR)

```
% myfir.m
function y = myfir(x)
%#codegen
persistent h idx buf
if isempty(h)
    h = [0.1 0.15 0.5 0.15 0.1];      % 예시 계수
    buf = zeros(1, numel(h));
    idx = int32(0);
end
y = zeros(size(x));
for n = 1:numel(x)
    idx = idx + 1;
    if idx > numel(h), idx = int32(1); end
    buf(idx) = x(n);
    % 순환버퍼 컨볼루션
    acc = 0.0;
    k = idx;
    for t = 1:numel(h)
        acc = acc + h(t)*buf(k);
        k = k - 1; if k < 1, k = numel(h); end
    end
    y(n) = acc;
end
```

### 2) C 코드 생성 (MATLAB Coder)

```
cfg = coder.config('lib','ecoder',false); % 정적/공유 라이브러리 생성
cfg.GenerateReport = true;
codegen -config cfg myfir -args {zeros(1024,1,'double')}
```

- `codegen` 후 `codegen/lib/myfir/` 폴더에 `myfir.h/.c` 등 생성.
- 이 산출물은 **MATLAB 불필요**.

### 3) DPI-C 어댑터 (C 래퍼)

```
// dpi_myfir.c
#include "svdpi.h"
#include "myfir.h"   // codegen 산출 헤더

// SV에서 실수 배열을 주고받기 위한 간단 래퍼
// 입력: real x[] (size N), 출력: real y[] (size N)
void dpi_myfir(const int N, const double* x, double* y) {
    // codegen된 entry에 맞춰 호출
    // MATLAB Coder가 만든 시그니처가 다르면 거기에 맞춰 수정
    // 예: myfir(double x[1024], double y[1024]) 형태일 수도 있음
    myfir(x, y); // 필요한 경우 반복/버퍼링 처리
}
```

### 4) SystemVerilog 선언/호출

```
// tb_top.sv
import "DPI-C" function void dpi_myfir(input int N,
                                       input  real x[],
                                       output real y[]);

module tb_top;
  int N = 1024;
  real xin [0:1023];
  real yout[0:1023];

  initial begin
    foreach (xin[i]) xin[i] = $urandom_range(-1000,1000)/1000.0;
    dpi_myfir(N, xin, yout);
    // yout를 DUT 출력과 비교 등
  end
endmodule
```

### 5) 빌드(예: VCS)

```
# 1) C 코드/헤더 경로 지정
VCS_INC=codegen/lib/myfir
VCS_C   ="dpi_myfir.c $VCS_INC/myfir.c $VCS_INC/myfir_data.c"

# 2) 컴파일/링크
vcs -full64 -sverilog tb_top.sv \
    -CFLAGS "-I$VCS_INC" \
    -cc gcc \
    -cpp g++ \
    -lstdc++ \
    $VCS_C -o simv

./simv
```

- **Questa/ModelSim**

  ```
  vlog +acc tb_top.sv
  gcc -fPIC -shared -I$QUESTA_HOME/include -I$VCS_INC \
      dpi_myfir.c $VCS_INC/myfir.c $VCS_INC/myfir_data.c -o dpi_myfir.so
  vsim -sv_lib dpi_myfir work.tb_top
  ```

- **Xcelium**

  ```
  xrun tb_top.sv dpi_myfir.c $VCS_INC/myfir.c $VCS_INC/myfir_data.c \
       -incdir $VCS_INC -sv
  ```

------

# 중요한 주의사항

1. **Coder vs Compiler**

- ✅ **MATLAB Coder/Embedded Coder**: **독립 C/C++ 생성 → MATLAB, MCR 불필요**
- ❌ **MATLAB Compiler**: MCR 필요 → DPI에 부적합

1. **지원 함수/언어 제약**

- Coder가 지원하는 MATLAB 문법/함수만 사용해야 함.
- `persistent`/고정길이/고정소수점(`fi`) 등은 잘 맞음.
- **동적크기/가변길이**는 설정과 메모리 정책이 필요.

1. **고정소수점(Fixed-Point)**

- `fi` 사용 시 **Fixed-Point Designer** 필요.
- 생성 코드는 여전히 **MATLAB 없이** 돌아갑니다(툴박스는 “코드 생성 시”에만 필요).

1. **상태/재진입성**

- `persistent` 상태를 쓰면 **스레드 안전성** 고려. 여러 인스턴스를 쓰려면 상태 구조를 외부화하거나 reentrant 설정.

1. **배열 레이아웃 & I/O**

- C는 row-major, MATLAB은 column-major지만 Coder가 올바른 C 시그니처를 생성합니다.
- DPI에서 **오픈 배열**(`real x[]`)은 편하지만, 시뮬레이터별 포인터 규칙을 맞추세요. 필요 시 `svOpenArrayHandle` API 사용.

1. **성능**

- 매 사이클 호출보다 **벡터 단위(블록)**로 호출이 훨씬 빠릅니다.
- 큰 모델은 **공유 라이브러리**로 빌드해 로드 시간을 줄이세요.

------

# 체크리스트

-  사용한 기능이 **MATLAB Coder 지원** 목록에 있는지 확인
-  `codegen`으로 **순수 C/C++ 산출** 생성
-  **DPI-C 래퍼** 작성(입출력형 맞추기)
-  시뮬레이터별 **컴파일/링크 옵션** 적용
-  SV `import "DPI-C"` 시그니처와 C 함수 원형 일치 확인
-  블록 단위 호출로 성능 최적화

------

원하시면, **실제 귀하의 MATLAB 함수(예: 이미지/신호처리 커널)** 를 받아서 **DPI-C 래퍼 + VCS/Questa/Xcelium 빌드 스크립트**까지 맞춤형으로 만들어드릴게요.

##### 

# Question 

다음 오류 발생하는데? ??? This assignment writes a 'double' value into a 'int32' type. Code generation does not support changing types through assignment. Check preceding assignments or input type specifications for type mismatches. Error in ==> myfir Line: 21 Column: 19 Code generation failed: View Error Report

###### ChatGPT said:

Thought for 16s

이 에러는 **int32 변수(`idx`)에 double 연산 결과를 대입**해서 나는 겁니다. (예: `idx = idx + 1;`에서 `1`이 double)
 MATLAB Coder는 **암시적 형 변환 대입을 금지**하므로, 산술/비교/인덱싱에서 **형을 일치**시켜야 합니다.

아래처럼 고치면 됩니다:

### 수정된 `myfir.m` (Coder OK)

```
function y = myfir(x)
%#codegen
% 입력 x는 double 벡터라고 가정
% codegen 시: -args {zeros(1024,1,'double')}

persistent h N buf idx
if isempty(h)
    h   = [0.1 0.15 0.5 0.15 0.1];   % double
    N   = int32(numel(h));           % tap 수: int32
    buf = zeros(1, double(N));       % 버퍼는 double, 길이 인자는 double
    idx = int32(0);                  % 인덱스 상태: int32
end

y = zeros(size(x));                  % 출력 double

for n = 1:numel(x)
    % int32 상수 사용 (암시적 변환 금지)
    idx = idx + int32(1);
    if idx > N
        idx = int32(1);
    end

    % 인덱싱 시 타입 일치
    buf(double(idx)) = x(n);

    % 컨볼루션
    acc = 0.0;
    k = idx;
    % 루프 카운터는 double로 두고, k 비교/갱신은 int32로 유지
    for t = 1:double(N)
        acc = acc + h(t) * buf(double(k));
        k = k - int32(1);
        if k < int32(1)
            k = N;
        end
    end
    y(n) = acc;
end
```

### 핵심 수정 포인트

- `idx = idx + 1;` → `idx = idx + int32(1);`
- `if idx > numel(h)` → `if idx > N` (그리고 `N = int32(numel(h))`)
- 인덱싱: `buf(double(idx))`처럼 **인덱스는 double**, 상태 계산은 **int32**로 분리
  - (Coder는 int32 인덱싱도 지원하지만, 혼용 시 가장 안전한 방법은 인덱싱 직전에 `double(...)` 캐스팅)
- 길이 인자(`zeros` 등)는 double 필요 → `double(N)`

### codegen 예시

```
cfg = coder.config('lib');
cfg.GenerateReport = true;
codegen -config cfg myfir -args {zeros(1024,1,'double')}
```

이렇게 하면 형변환 에러 없이 C 코드가 생성됩니다.
 원하시면, **int32 인덱싱을 끝까지 유지**하는 버전(인덱싱도 int32로)이나, **완전 재진입(reentrant) 상태 구조체화** 버전도 만들어 드릴게요.

##### 

# Question

windows용 modelsim이면 어떻게 해?

좋아요—**Windows용 ModelSim/Questa**에서 MATLAB 없이 **DPI-C**로 돌리는 방법을 한 번에 정리해드릴게요. (핵심은: C 코드 → DPI 래퍼 DLL → `vsim -sv_lib` 로드)

------

# 0) 전제**

- C 코드는 **MATLAB Coder/Embedded Coder**로 이미 뽑았다고 가정 (`myfir.c/.h`, `myfir_data.c` 등).
- ModelSim/Questa는 **64-bit** 설치라고 가정. (32/64 비트 반드시 일치해야 함)

------

# 1) SystemVerilog 선언 (open array 사용)

SV 쪽에서 **가변 길이 실수 배열**을 넘기면, C에서는 `svOpenArrayHandle`로 받는 게 정석입니다.

```
// tb_top.sv
import "DPI-C" function void dpi_myfir(input  real x[],  // 입력 벡터
                                       output real y[]); // 출력 벡터 (크기 x와 동일)

module tb_top;
  localparam int N = 1024;
  real xin [N], yout[N];

  initial begin
    foreach (xin[i]) xin[i] = $urandom_range(-1000,1000) / 1000.0;
    dpi_myfir(xin, yout);   // 길이는 C 쪽에서 svSize()로 알아냄
    // yout를 DUT와 비교 등...
  end
endmodule
```

------

# 2) DPI-C 래퍼 (Windows/ModelSim 방식)

`svdpi.h`의 **open array API**를 사용합니다. (길이: `svSize(h, 1)`, 포인터: `svGetArrayPtr1/2`)

```
// dpi_myfir.c
#include "svdpi.h"
#include "myfir.h"      // MATLAB Coder 산출물 헤더
#include "myfir_data.h" // 필요 시 포함

#ifdef __cplusplus
extern "C" {
#endif

__declspec(dllexport)   // Windows DLL export
void dpi_myfir(const svOpenArrayHandle x_h, const svOpenArrayHandle y_h)
{
    // 1) 길이 구하기 (1차원 배열)
    int N = svSize(x_h, 1);

    // 2) 포인터 얻기
    double* x = (double*) svGetArrayPtr(x_h);
    double* y = (double*) svGetArrayPtr(y_h);

    // 안전 체크
    if (!x || !y) return;

    // 3) MATLAB Coder로 생성된 엔트리를 호출
    //    Coder가 생성한 시그니처에 맞게 호출부를 맞추세요.
    //    예: myfir(const double x[1024], double y[1024]) 처럼 고정 크기일 수도 있습니다.
    //    그 경우 N이 코드젠 크기와 같을 때만 호출하거나,
    //    블록 분할/루프를 돌려서 맞춥니다.

    // (예시) N과 무관하게 처리 가능한 시그니처라고 가정:
    // myfir(x, y);
    // -- 만약 고정크기 함수면:
    // for (int i = 0; i < N; i += 1024) myfir(&x[i], &y[i]);

    myfir(x, y); // <- 실제 생성 함수에 맞게 수정

    // 상태를 쓰는 모델이면 필요 시 initialize/terminate 호출
    // myfir_initialize(); myfir_terminate();
}

#ifdef __cplusplus
}
#endif
```

> ✔️ **중요**: Coder가 만든 함수 원형이 프로젝트마다 다릅니다. `myfir.h`를 열어 정확히 맞춰주세요. 고정 크기이면 블록 단위로 나눠 부르거나, Coder 설정을 가변 길이로 바꿔 빌드하세요.

------

# 3) Windows에서 DLL 빌드 (MSVC)

1. **x64 Native Tools Command Prompt for VS**(Visual Studio) 실행
2. 환경변수(예시)

```
set MTI_HOME=C:\modeltech64_2023.4       rem 또는 %QUESTA_HOME%
set INC=%MTI_HOME%\include
set CGEN=codegen\lib\myfir               rem Coder 산출물 폴더
```

1. 컴파일/링크

```
cl /LD /I"%INC%" /I"%CGEN%" ^
  dpi_myfir.c "%CGEN%\myfir.c" "%CGEN%\myfir_data.c" ^
  /Fe:dpi_myfir.dll
```

- `/LD` = DLL 빌드
- `svdpi.h` 는 `%MTI_HOME%\include`에 있음
- **64-bit ModelSim**이면 반드시 **x64** 컴파일러 프롬프트에서 빌드하세요.
- C++라면 `extern "C"` 유지 + `__declspec(dllexport)` 필요.

------

# 4) ModelSim/Questa에서 로드 & 실행

```
vlog -sv tb_top.sv
vsim -64 work.tb_top -sv_lib dpi_myfir   rem .dll 확장자 없이 베이스 이름만
run -all
```

- GUI에서 하려면 `.do` 스크립트에 동일하게 작성:

```
vlib work
vlog -sv tb_top.sv
vsim -64 work.tb_top -sv_lib dpi_myfir
run -all
```

------

# 5) 흔한 에러/트러블슈팅

- **#1 32/64비트 불일치**: ModelSim이 64비트인데 32비트 DLL을 만들면 로드 실패. (반대도 동일)
- **#2 심볼 못 찾음**: 함수명이 맹글링 되었거나 export 안 됨 → `extern "C"` + `__declspec(dllexport)` 확인.
- **#3 svdpi.h 못 찾음**: `%MTI_HOME%\include` 경로 확인.
- **#4 런타임 DLL 누락**: MSVC 런타임 의존성 문제 → VS 설치본의 x64 프롬프트로 빌드하면 보통 해결.
- **#5 시그니처 불일치**: SV 선언과 C 프로토타입이 달라서 크래시 → open array는 `svOpenArrayHandle` 써야 안정적.
- **#6 고정 길이 생성물**: Coder가 `double x[1024]`로 만들었는데 SV에서 2048 넣음 → 크기 맞추거나 블록 호출로 분할.

------

# 6) 성능 팁

- **샘플 단위(매 사이클) 호출 금지**, **블록/프레임** 단위 호출 권장.
- 큰 모델은 초기화/종료를 분리(`*_initialize`, `*_terminate`)하고, 호출은 순수 계산만 수행.
- 가능하면 Coder에서 **Reentrant(재진입)** 설정 + 상태를 외부화해 멀티인스턴스 대비.

------

필요하시면, 지금 쓰시는 **`myfir.h`의 실제 시그니처**를 알려주세요. 그에 맞춘 **정확한 DPI 래퍼 코드**와 **.do 스크립트**를 바로 만들어드릴게요.