Python 으로 PDF를 JPG로 변환하려면, 아래 순서대로 따라 하시면 됩니다. 완전 초보여도 문제없어요 😊

------

## 1. Python 설치 (한 번만 하면 됨)

### 🔹 이미 설치돼 있다면 이 단계는 건너뛰세요!

- https://www.python.org/downloads/ 에서 Python 3.x 다운로드 후 설치
- 설치 중 **“Add Python to PATH”** 꼭 체크하세요 ✅

설치 확인:

```bash
python --version
```

------

## 2. 필요한 라이브러리 설치

터미널(cmd, PowerShell, Terminal 등)에서 아래 명령 실행:

```bash
pip install pdf2image pillow
```

------

## 3. Poppler 설치 (Windows만 해당)

`pdf2image`는 내부적으로 Poppler라는 도구를 사용합니다.

### 🔸 설치 방법:

1. 여기서 다운로드 👉 https://github.com/oschwartz10612/poppler-windows/releases
2. ZIP 파일 다운로드 후 압축 해제
3. 예를 들어 `C:\poppler-xx\Library\bin` 경로를 **환경 변수(Path)에 추가**

------

## 4. Python 코드 실행

### 🔸 예제 코드 저장

메모장이나 VSCode 등에 아래 코드 붙여넣고 `pdf_to_jpg.py`로 저장:

```python
from pdf2image import convert_from_path

images = convert_from_path("sample.pdf")  # 여기에 변환할 PDF 이름
for i, img in enumerate(images):
    img.save(f"page_{i + 1}.jpg", "JPEG")
```

📁 PDF 파일(`sample.pdf`)은 **이 Python 파일과 같은 폴더**에 두세요.

------

### 🔸 실행하기

터미널(cmd) 열고 해당 파일이 있는 폴더로 이동 후:

```bash
python pdf_to_jpg.py
```

성공하면 `page_1.jpg`, `page_2.jpg` 등의 이미지가 생성됩니다!

------

