OpenCV를 설치했는데 `mingw` 폴더가 없다는 건 **공식 OpenCV prebuilt 라이브러리에는 MinGW용 바이너리가 포함되지 않기 때문**입니다 😥 대부분은 **MSVC용(Visual Studio)** 빌드만 포함돼 있어요.

------

## 해결 방법 2가지 (선택)

### 방법 1: OpenCV를 직접 MinGW로 빌드 (정석, 강추🔥)

직접 빌드하면 정확히 내 환경에 맞는 라이브러리를 만들 수 있어요.

### 방법 2: GitHub에서 MinGW용 OpenCV 빌드된 버전 받기 (빠름, 쉬움⚡)

------

## 🔧 방법 1: OpenCV를 MinGW로 직접 빌드하기 (권장)

### 1. 필요한 도구 설치

- [CMake](https://cmake.org/download/)
- [MinGW-w64](https://www.mingw-w64.org/)
- Git (선택)

### 2. OpenCV 소스 다운로드

```bash
git clone https://github.com/opencv/opencv.git
git clone https://github.com/opencv/opencv_contrib.git  # 선택사항
```

### 3. 빌드 폴더 만들기

```bash
mkdir build
cd build
```

### 4. CMake 설정 (MinGW용)

```bash
cmake -G "MinGW Makefiles" -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=install ..
```

> `install` 폴더에 결과가 나옵니다.

### 5. 빌드

```bash
mingw32-make -j8   # CPU 쓰레드 수에 맞게 조절
mingw32-make install
```

📁 그럼 `install` 폴더에 MinGW용 OpenCV 라이브러리와 DLL이 생성돼요.

------

## 방법 2: 이미 빌드된 MinGW용 OpenCV 패키지 사용

### 👉 아래처럼 누군가 빌드해둔 저장소 이용 가능:

- https://github.com/huihut/OpenCV-MinGW-Build (비공식, 안정적)
- 또는 구글에 "OpenCV MinGW prebuilt" 검색

> 사용 시 반드시 **정확한 OpenCV 버전과 MinGW 버전이 일치해야 함**
>  (예: `OpenCV 4.5.1 + MinGW-w64 8.1.0` 등)

------

## 참고 팁

- Visual Studio 쓰면 OpenCV 연동이 훨씬 쉬움 (단, GCC 아님)
- MinGW 빌드 시 시간이 많이 걸릴 수 있음 (최대 수 시간)
- `opencv_world` 옵션으로 하나의 `.dll/.a`로 빌드 가능

------

