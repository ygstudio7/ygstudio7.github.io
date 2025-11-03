## **✅ 방법 2: MSYS2의 `autotools` 사용 (configure 문제 해결)**

`configure`가 작동하지 않는 이유는 `autoconf` 및 `automake`가 없기 때문일 수 있습니다.
 다음 명령어를 실행하여 `configure`를 사용할 수 있도록 환경을 설정하세요.

### **1. 필요한 패키지 설치**

```
sh


pacman -S autoconf automake libtool
```

### **2. `configure` 실행**

```
sh
autoreconf -fi
./configure --prefix=/mingw64 --enable-static --disable-shared
make
make install
```

이제 `configure`가 작동할 것입니다.







# libpng 설치

libpng를 정적 라이브러리로 만들어서 MSVCR80.dll 오류를 해결 가능

git-bash

```
git clone https://github.com/glennrp/libpng.git 

```

msys2 mingw64 터미널

```


cd libpng //cd _ygkim/synapse/202302_Imagniation_Supernova/a_test/libpng/

autoreconf -fi
./configure --prefix=/mingw64 --enable-static --disable-shared
make
make install

```





# zlib 설치



다음과 같은 오류가 나면 zlib를 설치한다.

/libpng/png.c:136:(.text+0xfe0): undefined reference to `crc32'`

MSYS2에서

`pacman -S zlib'