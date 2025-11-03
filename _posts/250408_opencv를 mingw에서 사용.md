좋아요! OpenCV를 **Windows + MinGW (GCC)** 환경에서 사용하는 방법을 순서대로 설명해드릴게요. 💡 조금 복잡하지만, 잘 따라오시면 성공할 수 있어요!

------

## 전체 요약

1. OpenCV Windows용 설치
2. MinGW 경로 설정
3. OpenCV와 연동하여 컴파일
4. DLL 복사 후 실행

------

## 1. OpenCV 다운로드 & 설치

### 📥 [OpenCV 윈도우용 prebuilt 다운로드 (zip)]

👉 https://github.com/opencv/opencv/releases

- 최신 **Windows .exe or .zip** 파일 다운로드
- 예: `opencv-4.9.0-windows.exe`
- 압축 해제 후 예시 경로: `C:\opencv`

------

## 2. MinGW 설치 및 경로 설정

OpenCV는 **MinGW 64bit** 기준으로 설명합니다.

1. https://www.mingw-w64.org/ 에서 설치
2. 설치 경로 예: `C:\mingw64`
3. 시스템 환경변수에 `C:\mingw64\bin` 추가

확인:

```bash
g++ --version
```

------

## 📂 3. OpenCV 헤더와 라이브러리 경로 지정

```bash
g++ main.cpp ^
  -IC:\opencv\build\include ^
  -LC:\opencv\build\x64\mingw\lib ^
  -lopencv_core490 -lopencv_imgcodecs490 -lopencv_highgui490 -lopencv_imgproc490
```

**⚠️주의**: 위에서 `-lopencv_core490` 같이 붙는 숫자는 버전마다 다릅니다.
 `C:\opencv\build\x64\mingw\lib` 폴더에서 `.a` 또는 `.dll.a` 파일 이름 보고 맞춰야 해요.

------

## 📌 예제 코드 (`main.cpp`)

```cpp
#include <opencv2/opencv.hpp>

int main() {
    cv::Mat img = cv::imread("image.jpg");
    if (img.empty()) {
        std::cout << "이미지 불러오기 실패!" << std::endl;
        return -1;
    }
    cv::imshow("Image", img);
    cv::waitKey(0);
    return 0;
}
```

------

## 🧪 4. 실행 전 DLL 복사

실행 파일이 뜨려면 OpenCV DLL 파일이 **같은 폴더**에 있어야 합니다.

- 예: `opencv_core490.dll`, `opencv_highgui490.dll` 등
- 경로: `C:\opencv\build\x64\mingw\bin`

✅ **실행 파일(.exe)과 같은 위치에 복사**하거나
 ✅ `PATH`에 `C:\opencv\build\x64\mingw\bin` 추가

------

## 🧠 팁

| 목적                        | 추천                                      |
| --------------------------- | ----------------------------------------- |
| 매번 g++ 옵션 입력 귀찮다면 | `Makefile`이나 `CMake` 사용               |
| OpenCV 전체 기능 사용       | `opencv_world490.dll` 하나로 통합 가능    |
| GUI 창 안 뜨면?             | `highgui` 링크 누락 확인 or DLL 누락 확인 |

------

## ✅ 확인용 빠른 테스트

```bash
g++ main.cpp -IC:\opencv\build\include -LC:\opencv\build\x64\mingw\lib -lopencv_core490 -lopencv_imgcodecs490 -lopencv_highgui490 -lopencv_imgproc490 -o app.exe
app.exe
```

------

필요하면 CMake 프로젝트 템플릿이나 MinGW 환경 자동 세팅 스크립트도 만들어 드릴 수 있어요 😊
 👉 지금 쓰고 계신 MinGW 경로나 OpenCV 버전 알려주시면 맞춤으로 도와드릴게요!