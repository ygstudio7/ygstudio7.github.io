# 전체 목표

- `myfir.m`(MATLAB) → **MATLAB Coder**로 C 생성(`myfir.c/.h`)
- 생성된 C 함수를 **DPI-C 래퍼(dpi_myfir.c)** 로 감싸 DLL로 빌드
- **Questa/ModelSim(Windows, 64bit)** 에서 **SystemVerilog TB(tb_top.sv)** 가 DLL을 로드해 호출
- 시뮬레이션에서 FIR 결과를 콘솔/파일로 확인

------

# 파일별 설명

## 1) MATLAB 함수: `myfir.m`

```
function y = myfir(x)
%#codegen
persistent h idx buf
if isempty(h)
    h = [0.1 0.15 0.5 0.15 0.1];  % FIR 계수(합=1 → DC gain=1)
    buf = zeros(1, numel(h));     % 순환버퍼(최근 샘플 저장)
    idx = int32(0);               % 버퍼 인덱스(상태)
end
y = zeros(size(x));
for n = 1:numel(x)
    idx = idx + 1;                         % 순환 인덱스 증가
    if idx > numel(h), idx = int32(1); end % 범위 벗어나면 1로 wrap
    buf(idx) = x(n);                       % 최신 샘플 저장

    % 순환버퍼 기반의 컨볼루션 누산
    acc = 0.0;
    k = idx;
    for t = 1:numel(h)
        acc = acc + h(t)*buf(k);
        k = k - 1;
        if k < 1, k = int32(numel(h)); end % 역방향 wrap
    end
    y(n) = acc;                             % 출력 기록
end
```

- **핵심**: 고정된 5-tap FIR. `persistent` 상태(`buf`,`idx`)를 사용하여 **프레임 간 상태 유지**.
- **타입 주의**: Coder가 싫어하는 암시적 형변환을 피하려면(권장) `idx` 증감/비교를 `int32`로 맞추는 버전도 사용 가능(지금 것도 R2016에선 생성 성공한 것으로 확인됨).
- **codegen 시그니처(고정 1024)**: `void myfir(const double x[1024], double y[1024]);`

## 2) DPI 래퍼: `dpi_myfir.c`

```
#include "svdpi.h"   // Questa/ModelSim 제공 헤더 (DPI API)
#include "myfir.h"   // MATLAB Coder 생성 헤더

#ifdef __cplusplus
extern "C" {
#endif

__declspec(dllexport) void dpi_myfir_init(void) {  // (선택) 초기화 엔트리
    myfir_init();                                  // Coder가 만든 init 호출
}

__declspec(dllexport)
void dpi_myfir_1024(const svOpenArrayHandle x_h,
                    const svOpenArrayHandle y_h) {
    int Nx = svSize(x_h, 1), Ny = svSize(y_h, 1);
    if (Nx != 1024 || Ny != 1024) return;         // 고정 길이 보호

    const double* x = (const double*) svGetArrayPtr(x_h);
    double*       y = (double*)       svGetArrayPtr(y_h);
    if (!x || !y) return;

    myfir(x, y);                                   // Coder 함수 직접 호출
}
#ifdef __cplusplus
}
#endif
```

- **역할**: SV의 **open array**(real[]) → C의 `double*`로 포인터 변환, 길이 체크, `myfir` 호출.
- **초기화**: `dpi_myfir_init()`은 필요 시 한 번 호출. (현재 `myfir_init()`을 사용)

## 3) SystemVerilog TB: `tb_top.sv`

```
import "DPI-C" function void dpi_myfir_init();
import "DPI-C" function void dpi_myfir_1024(input real x[], output real y[]);

module tb_top;
  localparam int N = 1024;
  real xin [N], yout[N];

  initial begin
    dpi_myfir_init();                      // (선택) 1회 초기화
    foreach (xin[i]) xin[i] = i / 1023.0;  // 정규화된 ramp 입력 [0,1]

    dpi_myfir_1024(xin, yout);             // DPI 호출

    // 눈으로 빠른 확인
    $display("=== FIR Output ===");
    for (int i = 0; i < 10; i++)
      $display("x[%0d] = %f, y[%0d] = %f", i, xin[i], i, yout[i]);

    #1 $finish;
  end
endmodule
```

- **포인트**: 입력이 커 보이면 이렇게 **정규화**해서 쓰면 직관적.
   (계수 합이 1 → DC gain=1이므로, 입력 스케일이 곧 출력 스케일)

## 4) DLL 생성 배치: `gen_dpi_dll_questasim.bat` (맨 아래 코드 설명됨)

- **목적**: Windows(64bit)에서 **MSVC**로 `dpi_myfir.dll` 생성.
- **환경 전제**: 반드시 **x64 Native Tools Command Prompt**에서 실행 (MSVC/SDK 경로 자동 설정).

핵심 단계 정리:

1. **MSVC 도구 확인**: `where cl`, `where link`

2. **Questa 경로 설정**:

   - `MTI_HOME=C:\intelFPGA_pro\24.2\questa_fe`
   - `MTINC=%MTI_HOME%\include` (여기 `svdpi.h`)
   - `MTLIB=%MTI_HOME%\win64` (Intel 패키지 구조. 일부 버전은 `win64\lib`에 `mtipli.lib`)

3. **MATLAB Coder 산출물 경로**: `CGEN=codegen\lib\myfir`

4. **Windows SDK 경로 자동 탐색**: `...Lib\<버전>\um\x64`, `...Lib\<버전>\ucrt\x64`

5. **컴파일/링크**:

   ```
   cl /LD /MD ^
     /I"%MTINC%" /I"%CGEN%" ^
     dpi_myfir.c "%CGEN%\myfir.c" ^
     /Fe:dpi_myfir.dll ^
     /link /MACHINE:X64 ^
       /LIBPATH:"%MTLIB%" mtipli.lib ^
       /LIBPATH:"%VCLIB%" ^
       /LIBPATH:"%SDKUM%" ^
       /LIBPATH:"%SDKUCRT%"
   ```

   - `/LD`: DLL, `/MD`: 동적 CRT
   - `mtipli.lib`: **DPI API import lib** (없으면 `mti.lib`일 수 있음)
   - `VCLIB`/`SDKUM`/`SDKUCRT`: `kernel32.lib`, `msvcrt.lib` 등 표준 라이브러리 경로

> ✅ **빌드 성공 출력 예**
>  `Creating library dpi_myfir.lib and object dpi_myfir.exp`
>  `Creating DLL dpi_myfir.dll` → `Build OK: dpi_myfir.dll`

## 5) 시뮬레이션 배치: `run_dpi_questasim.bat`

```
dumpbin /headers dpi_myfir.dll | findstr /i machine  & rem DLL이 x64인지 확인
vlog -sv tb_top.sv
vsim -c -64 work.tb_top -sv_lib dpi_myfir -do "run -all; quit"
```

- `-sv_lib dpi_myfir`: `dpi_myfir.dll` 로드(확장자 생략)
- `-64`: 64bit vsim. DLL도 64bit여야 함.
- 콘솔 출력으로 앞 10개 샘플 표시.

------

# 사용/검증 흐름 (한 번에)

1. MATLAB에서 C 생성 (한 번만)

   ```
   cfg = coder.config('lib'); cfg.GenerateReport = true;
   codegen -config cfg myfir -args {zeros(1024,1,'double')}
   ```

2. `gen_dpi_dll_questasim.bat` 실행 → `dpi_myfir.dll` 생성

3. `run_dpi_questasim.bat` 실행 → TB 빌드/실행, 콘솔에 결과 표시

**예상 콘솔 출력(일부)**

```
# === FIR Output ===
# x[0] = 0.000000, y[0] = 0.000000
# x[1] = 0.000977, y[1] = 0.000439
# x[2] = 0.001955, y[2] = 0.001416
# ...
# Simulation complete via $finish(1)
```

------

# 트러블슈팅 핵심

- **`svSize/svGetArrayPtr` unresolved** → `mtipli.lib`(또는 `mti.lib`) 링크/경로 확인, `svdpi.h`는 **Questa 것** 사용.
- **`MSVCRT.lib/LIBCMT.lib/kernel32.lib` not found** → **x64 Native Tools Prompt**에서 실행, VS C++/Windows SDK 설치/경로 확인.
- **DLL 로드 실패** → 32/64bit 불일치(둘 다 x64), DLL 위치/권한 확인.
- **길이 불일치** → `Nx/Ny` 체크 실패. TB의 배열 크기는 반드시 1024.
- **Intel 패키지의 라이브러리 위치** → `%MTI_HOME%\win64` 또는 `...\win64\lib` 확인 후 `/LIBPATH` 수정.

------

# 확장/응용 팁

- **파일로 전체 출력**: TB에서 `$fopen/$fdisplay`로 `fir_output.txt` 저장 → MATLAB/Excel로 비교 가능.
- **고정소수점 일치**: DUT가 정수/고정소수점이면, 입력/계수를 동일 Q포맷으로 스케일링(포화/반올림 정책도 맞추기).
- **가변 길이 지원**: `coder.typeof` + `emxArray_*` API 사용해 래핑(필요 시 별도 예제 제공 가능).





# gen_dpi_dll_questasim.bat 설명

좋아요 👍 지금 올려주신 `gen_dpi_dll_questasim.bat`은 **Windows 환경에서 MATLAB Coder로 생성한 C 코드(`myfir.c`)와 DPI 래퍼(`dpi_myfir.c`)를 컴파일하여 ModelSim/Questa에서 쓸 수 있는 DLL(`dpi_myfir.dll`)을 만드는 자동화 스크립트**입니다.
 아래에서 **라인별 동작**을 아주 자세히 설명드릴게요.

------

## 📘 전체 목적

> **MATLAB → C 코드 → DPI DLL**
>  → Questa/ModelSim에서 SystemVerilog와 연동 가능하게 하기.

------

## ⚙️ 전체 구조 요약

1. Visual Studio 빌드 환경 점검
2. ModelSim(Questa) 헤더/라이브러리 경로 설정
3. MATLAB Coder 산출물(`myfir.c`) 존재 확인
4. Windows SDK 버전 자동 탐색
5. `cl.exe`(MSVC 컴파일러)로 DLL 빌드 (`dpi_myfir.dll` 생성)

------

## 🔍 라인별 상세 설명

### ▣ 0) 환경 전제:

```
@echo off
rem ==== 반드시 x64 Native Tools Prompt에서 실행 ====
```

- **x64 Native Tools Command Prompt for VS**
   (Visual Studio 또는 Build Tools에서 제공)에서 실행해야 함.
- 그래야 `cl`, `link`, `lib`, `dumpbin` 등이 PATH에 포함되고
   `MSVCRT.lib`, `kernel32.lib` 같은 시스템 라이브러리 경로가 자동으로 잡힘.

------

### ▣ 1) MSVC(Visual C++) 컴파일러 확인

```
where cl >nul || (echo [ERROR] cl not found. Open x64 Native Tools Prompt.& exit /b 1)
where link >nul || (echo [ERROR] link not found. Open x64 Native Tools Prompt.& exit /b 1)
set VCLIB=%VCToolsInstallDir%\lib\x64
if not exist "%VCLIB%" (
  echo [ERROR] MSVC lib path not found: %VCLIB%
  exit /b 1
)
```

- `where cl` → MSVC 컴파일러(`cl.exe`)가 PATH에 있는지 확인
- `where link` → 링커 확인
- `%VCToolsInstallDir%`는 Visual Studio가 미리 설정한 환경변수
   (예: `C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Tools\MSVC\14.39.33519\`)
- 이 경로 아래의 `lib\x64` 폴더를 `%VCLIB%` 변수로 지정 → 후에 `/LIBPATH`에 사용
- 없으면 오류 출력 후 종료

------

### ▣ 2) Questa/ModelSim 경로 설정

```
set MTI_HOME=C:\intelFPGA_pro\24.2\questa_fe
set MTINC=%MTI_HOME%\include
set MTLIB=%MTI_HOME%\win64
```

- **MTI_HOME**: Questa 설치 경로(Intel FPGA 패키지 포함 버전)
- **MTINC**: DPI 헤더(`svdpi.h`) 위치
- **MTLIB**: Questa의 **DPI import library** (`mtipli.lib`)가 위치한 폴더

```
if not exist "%MTINC%\svdpi.h" (
  echo [ERROR] svdpi.h not found under %MTINC%
  exit /b 1
)
```

- `svdpi.h` 존재 확인 → 없으면 Questa 설치 경로 확인 필요.

------

### ▣ 3) MATLAB Coder 산출물 확인

```
set CGEN=codegen\lib\myfir
if not exist "%CGEN%\myfir.c" (
  echo [ERROR] %CGEN%\myfir.c not found. Did you run codegen?
  exit /b 1
)
```

- MATLAB Coder로 생성된 코드(`myfir.c`, `myfir.h`, `myfir_types.h`) 위치.
- 존재하지 않으면 `codegen`을 실행하지 않았다는 뜻 → 오류 메시지 후 종료.

------

### ▣ 4) Windows SDK 버전 자동 탐색

```
set SDKROOT=%WindowsSdkDir%
for /f "delims=" %%V in ('dir /b /ad "%SDKROOT%Lib" ^| findstr /r "10\.[0-9]*\.[0-9]*\.[0-9]*" ^| sort /r') do (
  set SDKVER=%%V
  goto :gotver
)
:gotver
set SDKUM=%SDKROOT%Lib\%SDKVER%\um\x64
set SDKUCRT=%SDKROOT%Lib\%SDKVER%\ucrt\x64
```

- **Windows SDK 경로**(보통 `C:\Program Files (x86)\Windows Kits\10\`)
- `Lib` 폴더 아래의 최신 버전(`10.0.26100.0` 등)을 찾아
  - `%SDKUM%` = “User Mode” 라이브러리 (`kernel32.lib`, `user32.lib` 등)
  - `%SDKUCRT%` = “Universal C Runtime” 라이브러리 (`ucrtbase.lib`, `msvcrt.lib` 등)
- 최신 SDK를 자동으로 골라 `/LIBPATH`에 넣음.

```
echo Using Windows SDK:
echo   UM   = %SDKUM%
if defined SDKUCRT echo   UCRT = %SDKUCRT%
```

- 디버깅용 출력.

------

### ▣ 5) 빌드 (DLL 생성)

```
cl /LD /MD ^
	/I"%MTINC%" /I"%CGEN%" ^
  dpi_myfir.c ^
  "%CGEN%\myfir.c" ^
  /Fe:dpi_myfir.dll ^
  /link /MACHINE:X64^
    /LIBPATH:"%MTLIB%" mtipli.lib ^
    /LIBPATH:"%VCLIB%" ^
    /LIBPATH:"%SDKUM%" ^
    /LIBPATH:"%SDKUCRT%"
```

- `/LD` → DLL 생성
- `/MD` → 동적 CRT (msvcrt.dll 링크)
- `/I` 옵션 → include 경로 추가 (`svdpi.h`, `myfir.h` 등 포함)
- 입력 파일: `dpi_myfir.c`, `myfir.c`
- `/Fe` → 출력 DLL 이름 지정
- `/link` → 링커 옵션 시작
- `/MACHINE:X64` → 64비트 타깃 지정
- `/LIBPATH` → 링커가 `.lib` 파일을 찾을 폴더 지정
  - `mtipli.lib`: Questa의 DPI API
  - `%VCLIB%`: Visual Studio 런타임
  - `%SDKUM%`/`%SDKUCRT%`: Windows SDK 라이브러리

빌드 성공 시:

```
Microsoft (R) C/C++ Optimizing Compiler...
Creating library dpi_myfir.lib and object dpi_myfir.exp
Creating DLL dpi_myfir.dll
Build OK: dpi_myfir.dll
```

실패 시(에러코드 비정상):

```
if errorlevel 1 goto :e
echo.
echo Build OK: dpi_myfir.dll
exit /b 0

:e
echo.
echo [ERROR] Build failed. Check that VS C++ Build Tools and Windows SDK are installed.
exit /b 1
```

------

## 🧩 결과

| 파일                     | 생성 위치            | 설명                                                  |
| ------------------------ | -------------------- | ----------------------------------------------------- |
| `dpi_myfir.dll`          | 현재 폴더            | ModelSim/Questa가 `-sv_lib` 옵션으로 로드하는 DPI DLL |
| `dpi_myfir.lib` / `.exp` | 현재 폴더            | 링커 중간 산출물 (삭제해도 무방)                      |
| `myfir.c/.h`             | `codegen\lib\myfir`  | MATLAB Coder 산출물                                   |
| `svdpi.h`                | `%MTI_HOME%\include` | DPI API 헤더                                          |

------

## ✅ 빌드 완료 후 실행 단계

이 스크립트를 성공적으로 실행하면 **DLL 빌드 성공** 메시지가 뜨고,
 다음 스크립트로 시뮬레이션을 실행합니다:

```
dumpbin /headers dpi_myfir.dll | findstr /i machine  & rem 64비트 확인
vlog -sv tb_top.sv
vsim -c -64 work.tb_top -sv_lib dpi_myfir -do "run -all; quit"
```

------

## 🚀 요약

| 구분           | 설명                                                         |
| -------------- | ------------------------------------------------------------ |
| **역할**       | MATLAB C 코드 + DPI 래퍼를 컴파일하여 Questa용 DLL 생성      |
| **컴파일러**   | MSVC (`cl`, `link`)                                          |
| **헤더**       | `svdpi.h` (Questa), `myfir.h` (MATLAB Coder)                 |
| **라이브러리** | `mtipli.lib` (DPI API), `MSVCRT.lib` 등 (SDK)                |
| **출력**       | `dpi_myfir.dll`                                              |
| **필수 환경**  | ① x64 Native Tools Prompt ② VS C++ Build Tools ③ Windows SDK ④ Questa/ModelSim 설치 |

------

원하신다면 이 배치 스크립트에 **자동 시뮬레이션(run_dpi_questasim.bat)** 을 이어붙여
 “빌드 → 실행 → 결과 자동 출력”까지 한 번에 돌게 만들어드릴 수도 있습니다.
 그렇게 합쳐드릴까요?

