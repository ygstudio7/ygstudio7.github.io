msys2를 사용한 gcc설치

1. ## Installation

2. 1. Download the installer: [msys2-x86_64-20250221.exe](https://github.com/msys2/msys2-installer/releases/download/2025-02-21/msys2-x86_64-20250221.exe)

      (Optional) For more information on the installer, like command line options, or how to verify the checksum and signature of the installer, see the [installer guide](https://www.msys2.org/docs/installer/).

   2. Run the installer. Installing MSYS2 requires 64 bit Windows 10 or newer.

   3. Enter your desired **Installation Folder** (short ASCII-only path on a NTFS volume, no accents, no spaces, no symlinks, no subst or network drives, no FAT).

      ![Second screen of MSYS2 installation](https://www.msys2.org/images/install-2-path-dark.png#gh-dark-mode-only)

   4. When done, click **Finish**.

      ![Third screen of MSYS2 installation](https://www.msys2.org/images/install-3-finish-dark.png#gh-dark-mode-only)

      

3. 

4. ## **2. 패키지 데이터베이스 업데이트**

5. **터미널(MSYS2 MinGW 64-bit)**에서 다음 명령을 실행하여 패키지를 최신 상태로 만듭니다.

6. ```
   sh
   
   
   CopyEdit
   pacman -Syu
   ```

7. 업데이트 후 다시 MSYS2를 실행하고, 아래 명령어를 실행합니다.

8. ```
   sh
   
   
   CopyEdit
   pacman -Su
   ```

9. 

10. ## **3. MinGW 설치**

11. MSYS2에서는 여러 종류의 MinGW 환경을 제공합니다. 주로 사용하는 것은 **UCRT64**와 **MINGW64**입니다.

12. ### **(1) UCRT64 환경에서 MinGW 설치**

13. ```
    sh
    
    
    CopyEdit
    pacman -S mingw-w64-ucrt-x86_64-gcc
    ```

14. ### **(2) MINGW64 환경에서 MinGW 설치**

15. ```
    sh
    
    
    CopyEdit
    pacman -S mingw-w64-x86_64-gcc
    ```

16. ### **(3) MinGW 개발 도구 전체 설치**

17. Clang 및 기타 도구도 함께 설치하려면:

18. ```
    sh
    
    
    CopyEdit
    pacman -S mingw-w64-x86_64-toolchain
    ```

19. 

20. ## **4. 환경 변수 설정 (선택 사항)**

21. MinGW를 어디서나 사용하려면 환경 변수(`Path`)를 설정해야 합니다.

22. 1. **경로 확인**
       - UCRT64: `C:\msys64\ucrt64\bin`
       - MINGW64: `C:\msys64\mingw64\bin`
    2. **시스템 환경 변수 추가**
       - `제어판 > 시스템 > 고급 시스템 설정 > 환경 변수`로 이동
       - `Path` 변수에 위의 경로를 추가

23. ------

24. ## **5. 설치 확인**

25. 터미널에서 다음 명령어를 실행하여 MinGW가 정상적으로 설치되었는지 확인합니다.

26. ```
    sh
    
    
    CopyEdit
    gcc --version
    ```

27. 버전 정보가 출력되면 MinGW가 정상적으로 설치된 것입니다!

28. 

29. 

30. ## **3. make 설치**

31. MSYS2에서는 **make**를 다음과 같이 설치할 수 있습니다.

32. ### **(1) MinGW 환경에서 make 설치**

33. (주로 MinGW64 환경을 사용하는 경우)

34. ```
    sh
    
    
    CopyEdit
    pacman -S mingw-w64-x86_64-make
    ```

35. 설치 후에는 `mingw32-make`라는 이름으로 실행됩니다.

36. ### **(2) MSYS 환경에서 make 설치**

37. (MSYS 기본 쉘을 사용하는 경우)

38. ```
    sh
    
    
    CopyEdit
    pacman -S make
    ```

39. # png 설치

40. ```
    $ pacman -S mingw-w64-ucrt-x86_64-libpng
    ```

41. 

42. ### **(2) vcpkg 사용 (Visual Studio 환경)**

43. 1. **vcpkg 다운로드 및 설치**

       ```
       shCopyEditgit clone https://github.com/microsoft/vcpkg.git
       cd vcpkg
       .\bootstrap-vcpkg.bat
       ```

    2. **libpng 설치**

       ```
       sh
       
       
       CopyEdit
       .\vcpkg install libpng
       ```

    3. **CMake 또는 Visual Studio에서 vcpkg 경로 설정**

       - CMake 사용 시:

         ```
         sh
         
         
         CopyEdit
         cmake -DCMAKE_TOOLCHAIN_FILE=[vcpkg 경로]\scripts\buildsystems\vcpkg.cmake ..
         ```

       - Visual Studio에서는 `vcpkg integrate install` 명령 실행





# 참고: 

https://chatgpt.com/c/67eda210-1b14-800b-aa0f-622347961918



# 주의

g++이 설치되지 않으면, Mcafee의 실시간 감시를 잠시 15분 정도 중지한다.

![image-20250404191054583](assets/image-20250404191054583.png)