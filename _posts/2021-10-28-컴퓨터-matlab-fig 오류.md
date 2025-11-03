MATLAB 은 20년 이상 오랫동안 사용되어온 프로그램으로 버전이 올라가면서 MAT 파일을 저장하는 방식도 수 차례 변경되었다. 

만약, GUI 프로그램을 만들기 test.fig 파일을 신규 MATLAB 버전에서 만들고, 오래된 MATLAB버전에서 읽고자 할 경우 다음과 같이 `hgload` 코드가 실행중에 `invalid Figure file format`이라는 메시지가 출력된다. 

```matlab
??? Error using ==> hgload at 53
invalid Figure file format

Error in ==> openfig at 72
    [fig, savedvisible] = hgload(filename, struct('Visible','off'));
```



이 메시지는 `test.fig` 파일의 포맷이 현재 MATLAB 버전에서는 읽을 수 없다는 것을 의미한다.

아래 MATLAB 기본 설정을 선택한 후, 좌측에서 `MATLAB > 일반 > MAT-file`을 선택하면, 우측에 MAT-file 저장 형식이 나온다. 만약 기존 MATLAB에서 MAT나 fig 파일들이 읽혀지지 않는다면, `MATLAB Version 5이상`을 선택해서 저장해야 한다. 아래 참고사항에서 보듯이 플롯 및 GUI를 저장하여 만드는 FIG 파일에도 적용된다고 나와있다.

![image-20211028175513181](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-matlab-fig 오류.assets\image-20211028175513181.png)

