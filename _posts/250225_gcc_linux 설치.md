Windows에서 Linux 환경을 설정하고 GCC(GNU Compiler Collection)를 설치하는 방법은 여러 가지가 있습니다. 가장 일반적인 방법은 다음과 같습니다.  

---

## 1. **WSL(Windows Subsystem for Linux) 사용** (추천)  

Windows에서 공식적으로 제공하는 WSL을 사용하면 Linux 환경을 직접 실행할 수 있습니다.  

### **설치 방법**  

1. **WSL 활성화**  

   - Windows 검색에서 "PowerShell"을 검색하고 "관리자 권한으로 실행"을 선택합니다.  

   - 다음 명령을 입력하여 WSL을 활성화합니다.  

     ```powershell
     wsl --install
     ```

   - 시스템을 재부팅합니다.  

2. **Ubuntu 또는 다른 Linux 배포판 설치**  

   - 명령 프롬프트(CMD) 또는 PowerShell에서 다음을 실행합니다.  

     ```powershell
     wsl --install -d Ubuntu
     ```

   - 설치 후 Ubuntu 터미널이 열리면, 사용자 이름과 비밀번호를 설정합니다.  

3. **GCC 설치**  

   - WSL 터미널에서 다음 명령을 실행하여 GCC를 설치합니다.  

     ```bash
     sudo apt update
     sudo apt install build-essential -y
     ```

   - 설치 완료 후 GCC 버전 확인:  

     ```bash
     gcc --version
     ```

---

## 2. **MinGW-w64 사용** (WSL 없이 Windows에서 직접 실행)  

MinGW-w64는 Windows에서 네이티브로 GCC를 실행할 수 있도록 지원하는 도구입니다.  

### **설치 방법**  

1. **MinGW-w64 다운로드 및 설치**  
   - [Mingw-w64 공식 사이트](https://www.mingw-w64.org/) 또는 [Winlibs](https://winlibs.com/)에서 최신 버전을 다운로드합니다.  
   - 설치 중 `gcc` 및 `g++` 컴파일러를 선택합니다.  

![image-20250225104302427](./assets/image-20250225104302427.png)

![image-20250225104214222](./assets/image-20250225104214222.png)

1. **환경 변수 설정**  

   - `C:\mingw-w64\bin` (설치 경로) 을 환경 변수(`PATH`)에 추가합니다.  

   - `cmd`를 열고 다음을 입력하여 GCC가 제대로 설치되었는지 확인합니다.  

     ```cmd
     gcc --version
     ```

---

## 3. **MSYS2 사용**  

MinGW보다 더 유연한 Windows용 패키지 관리 시스템입니다.  

### **설치 방법**  

1. **MSYS2 다운로드 및 설치**  

   - [MSYS2 공식 사이트](https://www.msys2.org/)에서 설치 파일을 다운로드합니다.  
   - 설치 후 `MSYS2 MSYS` 터미널을 실행합니다.  

2. **패키지 업데이트**  

   ```bash
   pacman -Syu
   ```

3. **GCC 설치**  

   ```bash
   pacman -S mingw-w64-x86_64-gcc
   ```

4. **환경 변수 설정**  

   - `C:\msys64\mingw64\bin`을 환경 변수(`PATH`)에 추가합니다.  
   - `cmd` 또는 `PowerShell`에서 `gcc --version`을 실행하여 확인합니다.  

---

## 4. **Cygwin 사용** (대체 방법)  

Cygwin은 Windows에서 Linux 환경을 흉내 내는 도구입니다. 하지만 WSL보다 비효율적이므로 추천되지 않습니다.  

### **설치 방법**  

1. [Cygwin 다운로드](https://www.cygwin.com/)  

2. 설치 중 `gcc-g++` 패키지를 선택하여 설치  

3. Cygwin 터미널에서 GCC 확인  

   ```bash
   gcc --version
   ```

---

## **어떤 방법을 선택해야 할까?**  

- **WSL(Windows Subsystem for Linux)** → 리눅스 개발 환경을 원한다면 (추천)  
- **MinGW-w64** → Windows에서 간단한 C/C++ 프로그램을 컴파일하려면  
- **MSYS2** → MinGW보다 더 많은 패키지 관리가 필요하면  
- **Cygwin** → Linux 환경이 필요하지만 WSL을 사용할 수 없을 때  

