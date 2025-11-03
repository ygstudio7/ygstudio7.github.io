DOS 커맨드 창에서 환경변수를 설정하는 방법을 알아본다.

윈도우 키 + R을 눌러서 실행창을 열고, cmd를 입력하여 DOS 커맨드 창을 오픈한다.



![image-20210617110034633](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-06-17-컴퓨터-cmd에서 path 설정.assets\image-20210617110034633.png)



`path`를 입력하면, 현재 환경변수 정보를 보여준다. `echo %path%` 도 동일한 정보를 출력한다. 

```dos
> path 
> echo %path%
```



`set path` 명령을 사용하여, 현재 환경변수의 path 정보인`%path%`에 `;`을 사용하여 원하는 경로를 추가할 수 있다.

```
> set path = %path%; <추가할 경로>
```





https://kuroikuma.tistory.com/33

