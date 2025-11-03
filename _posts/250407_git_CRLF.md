"git 설치 후 ^M 차이"라고 하신 게 아래와 같은 경우를 말씀하시는 걸까요?

- Git 저장소에 있는 파일을 `git clone` 하거나 작업하면서, 파일 내용 중에 `^M` 문자가 생기는 경우
- 또는 `^M` 차이가 `git diff`에 나타나는 경우

이건 주로 **줄바꿈(LF/CRLF)** 문제 때문입니다.

------

### 🔍 `^M` 문자의 정체는?

- `^M`은 `CR` (Carriage Return, ASCII 13, `\r`) 문자를 뜻합니다.
- Windows는 줄 끝을 `CRLF` (`\r\n`)로 저장하지만,
- Linux/macOS는 `LF` (`\n`)만 사용합니다.

그래서 파일을 Windows에서 만들고 Git을 통해 Linux/macOS에서 열거나 반대의 경우, `^M` 문자가 보일 수 있어요.

------

### ✅ 해결 방법

#### 1. **Git의 자동 줄바꿈 설정 확인하기**

```bash
git config --global core.autocrlf
```

- `true`: 체크아웃 시 `LF` → `CRLF`, 커밋 시 `CRLF` → `LF`
- `input`: 체크아웃 시 아무것도 안 하고, 커밋 시 `CRLF` → `LF`
- `false`: 변환하지 않음

**권장 설정 (Linux/macOS 사용 시):**

```bash
git config --global core.autocrlf input
```

**Windows 사용 시:**

```bash
git config --global core.autocrlf true
```

#### 2. **파일에서 직접 ^M 제거하기**

```bash
# macOS/Linux에서
sed -i 's/\r//' filename
```

------

### 💡 `git diff`에서 ^M만 차이나는 경우?

그럼 `git diff`가 줄 끝 문자 차이를 보여주는 거예요. 아래처럼 diff를 무시하게 할 수 있어요:

```bash
git diff --ignore-space-at-eol
```

또는 `.gitattributes` 파일에 아래 설정을 추가해 강제로 LF만 쓰도록 할 수도 있어요:

```bash
*.txt text eol=lf
*.sh text eol=lf
```

------

필요하시면 OS별 설정법이나 `.gitattributes` 예시도 드릴 수 있어요. 혹시 어떤 상황에서 `^M`을 보셨는지 조금만 더 설명해주실 수 있을까요?

# 내 설정

git config --global core.autocrlf true

나중에 아래처럼할 수 도 있다. 그리고 CRLF를 LF만 저장되도록 할 수 도 있다 

git config --global core.autocrlf input