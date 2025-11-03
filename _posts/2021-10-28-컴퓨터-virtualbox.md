

![image-20211028105303913](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028105303913.png)

VirtualBox는 윈도우, 리눅스, 매킨토시, 솔라리스 OS에서 동작하는 프로그램으로, 다양한 guest OS를 해당 host OS에서 사용할 수 있도록 지원해준다. 사용가능한 guest OS로는 NT 4.0, 2000, XP, Server 2003, Vista, Windows 7, Windows 8, Windows 10), DOS/Windows 3.x, Linux (2.4, 2.6, 3.x and 4.x), Solaris and OpenSolaris, OS/2, 그리고 OpenBSD  등이 있다.



# 설치 파일 다운로드

구글에서 virtualbox를 검색한다.

검색결과에서 가장 먼저 나오는 Oracle VM VirtualBox (https://virtualbox.org)홈페이지에 들어간다.

![image-20211028105234958](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028105234958.png)



홈페이지 중앙의 `Download VirtualBox 6.1`버튼을 클릭하여 설치파일을 받는다.

![image-20211028105340964](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028105340964.png)



현재 최신 버전은 VirtualBox 6.1.28으로, `Windows hosts`를 선택해서 설치파일(*.exe)을 다운로드한다.

![image-20211028105756150](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028105756150.png)



# 설치하기

설치파일을 실행하면 다음과 같이 환영 메시지 창이 출력되고, `Next` 버튼을 누른다.

![image-20211028105924962](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028105924962.png)

설치 할 내용들과 설치 폴더를 선택하는 창에서는 변경사항없이 `Next` 버튼을 누른다.

![image-20211028110013307](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028110013307.png)

옵션 선택창에서도, 변경없이 `Next`버튼을 누른다.

![image-20211028110045193](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028110045193.png)

설치 진행여부를 묻는 창에서도 `Yes`버튼을 눌러서 진행한다.

![image-20211028110141838](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028110141838.png)

설치 과정이 진행되고, 

![image-20211028110218529](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028110218529.png)

이후, 장치 소프트웨어를 설치할지 여부를 묻는 창이 나온다. 

범용직렬 버스 콘트롤러를 설치할지 묻는 창에서는 `설치`버튼을 눌러 진행한다.

![image-20211028110457401](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028110457401.png)

설치 완료 안내창이 나오면, `Finish` 버튼을 누르고 종료한다.

![image-20211028110535132](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028110535132-16354443355681.png)



이후, Oracle VM VirtualBox 관리자가 실행된다.

![image-20211028110617354](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028110617354.png)



# 기존 guest OS 가져오기

기존에 사용하던 guest os를 가져오는 경우, `머신 > 새로 만들기`를 선택한다.



![image-20211028110850124](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028110850124.png)



아래와 같이 가상 먼신 만들기 창에서는, `이름`에 사용할 운영 체제의 이름을 선택하고, 종류와 버전을 선택한 후, `다음` 버튼을 누른다.

![image-20211028110928577](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028110928577.png)



메모리 크기는 윈도우의 경우 최소한 2GB (2048 MB) 정도는 필요하고, 메모리를 많이 사용한 경우에는 4GB - 8GB까지도 설정하는게 좋다.

![image-20211028111150059](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028111150059.png)

가상 머신 만들기 창에서는 [o] 기존 가상하드 디스크 파일 사용을 선택하고, 우측의 폴더 아이콘을 선택해서, 기존 가상 머신 (*. vdi) 파일을 찾아서 선택한 후, `만들기`버튼을 누른다.

![image-20211028111328974](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028111328974.png)

새로운 가상 머신이 추가되었고, 관련 정보들이 우측에 표시된다. 가상 머신을 실행하기 위해서, 우측 상단에 보여지는 화살표 모양의 `시작` 아이콘을 클릭한다.

![image-20211028111623043](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028111623043.png)





# 유용한 설정 팁



### 화면 확장

부팅후 초기에 작은 화면을 확장하기 위해서는 상단 메뉴에서 `장치 > 게스트 확장 CD 이미지 삽입`을 선택해 주어야 한다. 



![image-20211028155000693](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028155000693.png)



다음과 같이 VBoxWindowsAdditions.exe를 자동 실행한다는 메시지 창이 출력되고, 선택해서 설치를 진행한다.

![image-20211028155222709](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028155222709.png)

이 프로그램이 컴퓨터를 변경할 수 있도록 `예` 버튼을 눌러 진행한다.

![image-20211028155338463](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028155338463.png)



처음 셋업 창에서 `Next`를 선택한다.

![image-20211028155403876](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028155403876.png)

설치할 폴더를 묻는창에서는 변경없이 `Next`를 선택한다.

![image-20211028155503109](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028155503109.png)

설치 요소를 묻는 창에서도 변경없이 `Install` 버튼을 선택한다.

![image-20211028155535604](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028155535604.png)

다음으로 Oracle Corporation 시스템 장치를 설치할지 묻는 창에서는 `설치`버튼을 선택한다.

![image-20211028160123795](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028160123795.png)

설치 완료된 후에는 [v] Reboot now를 선택하고 `Finsih` 버튼을 눌러서, 윈도우를 다시 시작합니다.

![image-20211028163949413](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028163949413.png)



### 클립보드 공유

두 OS간에 복사 밑 붙이기 등의 기능을 사용하려면, 클립보드를 공유해야 합니다.

상단 메뉴에서 `장치` > `클립보드 공유` > `양방향`을 선택합니다.

![image-20211028170809749](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-28-컴퓨터-virtualbox.assets\image-20211028170809749.png)



### 공유 폴더





### 드래그 앤 드롭

