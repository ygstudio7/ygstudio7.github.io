참고: https://tttsss77.tistory.com/76



# VMware windows 공유폴더 설정하기



본 글에서는 윈도우즈에 설치된 VMware 내에서 동작하는Guest 운영체제와 윈도우즈간에 공유폴더를 설정하는 방법을 소개한다.



본 글에서 사용된 환경은 다음과 같다.

- PC: Windows 10 Professional x64
- VMware: VMware Workstation 16 Pro
- VMware Guest 운영체제: Winows 7 x64



VMware 실행후 Guest 운영체제가 꺼진 상태에서, `VM` -> `Settings`을 선택하여, `Virtual Machine Settings` 창이 나오도록 한다.

![image-20210515115155037](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210515115155037.png)

![image-20210515115240533](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210515115240533.png)



`Options`탭으로 이동하여 `Shared Folders`를 선택한 후, `Folder sharing`을 `Always enabled`로 선택하고, `Folders`에서 `Add` 버튼을 선택하여, 공유폴더 마법사를 실행한다.

![image-20210515115542610](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210515115542610.png)



![image-20210515115618657](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210515115618657.png)





윈도우즈에서 공유할 폴더 경로를 선택하고, 마법사를 종료한다.

![image-20210515115825251](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210515115825251.png)



VMware의 Windows 에서 공유 폴더를 확안해보면 `Network` 상에서 `vmware-host` 폴더내에 위에서 공유한 폴더를 확인할 수 있다.

![image-20210515120429193](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210515120429193.png)

