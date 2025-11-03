

# 기존 라이선스 비활성화

기존 MATLAB 설치 폴더 혹은 MATLAB 메뉴에서 `Deactive MATLAB R2008b`를 선택합니다.

![image-20211007152707062](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007152707062.png)

만약 MATLAB 설치한 폴더나 드라이브가 분리되어 있다면, 해당 폴더내의 `uninstall` 폴더와 `licenses` 폴더만 있으면, 비활성화시킬 수도 있습니다. 해당 폴더들을 원래 설치되었던 드라이브의 설치 위치 (예를 들면, C:/Program Files/MATLAB/R2008b)와 동일한 폴더를 만들어 파일들을 복사한 후,

`deactivate_matlab.exe` 을 실행하면 예전 라이선스를 비활성화할 수 있습니다.

![image-20211007153043779](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007153043779.png)



만약 서버에 연결되지 않은 경우, 다음과 같이 Deactivation string이 화면에 보여지고, 

이 값을 MATLAB 라이선스 센터 홈페이지(https://www.mathworks.com/licensecenter)에 들어가서, 비활성화시킬 수도 있습니다.



![image-20211007153107917](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007153107917.png)



MATLAB 홈페이지의 라이선스 센터(https://www.mathworks.com/licensecenter)에 들어가면, 아래와 같이 현재 활성화된 라이선스들을 확인할 수 있습니다. 그 중에서 비활성화시킬 라이선스를 선택합니다. 

![image-20211007142003527](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007142003527.png)

아래와 같이 4개의 탭이 보인다. 비활성화를 위해서는 3번째 `Install and Activate`를 선택합니다.

![image-20211007142119270](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007142119270.png)



![image-20211007142250877](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007142250877.png)



관련된 라이선스 정보들을 확인할 수 있습니다. Activation Label과 Host ID는 해당 컴퓨터의 host ID가 설정되고, Activation For에 누가 활성화했는지 알 수 있습니다. 

Get License File을 누르면, 좌측의 화살표 아이콘은 License file을 다운로드할 수 있고 우측의 우편봉투 아이콘은 이메일로 받을 수 있습니다. 

![image-20211007142506625](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007142506625.png)

좌측의 `Download License File`화살표를 누르면 다음과 같이 해당 제품을 선택할 수 있습니다. 제품 선택후 `Continue`버튼을 누릅니다.

![image-20211007142925855](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007142925855.png)



해당 제품에 대해, `Download License File`버튼을 눌러서 라이선스 파일을 받을 수도 있고, 다른 컴퓨터에 설치할 때 필요한 File Installation Key도 확인할 수 있습니다. 

![image-20211007143007779](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007143007779.png)



비활성화를 위해, 우측의 `Deactivate`아이콘을 누릅니다. 한번더 확인하기 위한 확인창에서 `Deactive`버튼을 누릅니다. 아래와 같이 최종 비활성화되었다는 메시지가 나옵니다.

![image-20211007150058803](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007150058803.png)

![image-20211007150147459](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007150147459.png)



# 라이선스 활성화

새 컴퓨터에서 라이선스를 활성화시키기 위해서는,라이선스 센터에 들어가서 `Install and Active` 탭에서 `Active to Retrive License File`을 선택합니다.

![image-20211007150628699](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007150628699.png)



아래 창에서는 Host ID, Activation Label을 입력하고, `Continue`버튼을 누른다. 만약, Host ID를 모르는 경우 아래 `Host ID 확인`내용을 참고헙나다. Activation Label은 개별 라이선스를 확인하기 위한 것으로 Host ID를 입력해도 됩니다.

![image-20211007150655408](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007150655408.png)



아래와 같이 해당 컴퓨터에 MATLAB 설치시 사용할 수 있는 License File을 다운로드할 수 있는 창이 나옵니다. 좌측의 `Download License File`을 선택하면, 설치시 사용할 수 있는 `license.lic`파일을 받을 수 있고, Activation을 위한 File Installation Key도 확인할 수 있습니다. 

![image-20211007151757532](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007151757532.png)



### Host ID 확인

Host ID는 컴퓨터를 유일하게 인식하는 특별한 정보중 하나로서, MATLAB 라이선스 파일을 생성하기 위해 사용됩니다. Host ID는 보통 MAC address가 사용되기고 하지만, Windows마다 개발 license를 가진 경우에는, C 드라이브의 volume serial 번호가 host ID로 사용됩니다. 

윈도우즈에서 Host ID를 알아내기 위해서는 커맨드 창을 실행하고 다음과 같이 C 드라이브의 volume serial을 얻는 명령을 입력합니다.

```bash
vol c:
```



### Username 확인

MATLAB을 개별 라이선스로 설치하기 위해서는 username이 필요합니다. 

username을 확인하는 가장 쉬운 방법중 하나는 커맨드 창을 열고, 다음과 같이 `set`명령을 사용하여 username을 얻는 것입니다.

```bash
set username
```





# MATLAB 다운로드

MATLAB 사용자는 MATLAB 다운로드 페이지 (https://www.mathworks.com/downloads/)에서 직접 설치파일들을 받을 수 있습니다.

좌측 `Select Release`메뉴에서 라이선스를 가진 MATLAB을 선택하면, 우측과 같이, 본인 라이선스로 받을 수 있는 설치파일과 MATLAB, Toolbox들이 표시됩니다.

각 `Download`버튼을 눌러, 이들을 한 폴더(예를 들면, `C:\Temp\`)에 다운로드합니다.



![image-20211007153746255](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007153746255.png)



# MATLAB 설치

필요한 파일들을 다 받은 후에는, `Installer.exe`파일을 눌러서 실행시키고, 압축 파일들을 풀어서 설치를 시작합니다.

이미 라이선스를 활성화하고, `license.lic`파일과 `File Activation Key`알아놨으니, 이제 인터넷 없이 수동 설치를 선택하고, `Next`버튼을 누릅니다.

![image-20211007154015611](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007154015611.png)

라이선스 동의 창에서 `Yes`를 선택하고, `Next`버튼을 누릅니다.

![image-20211007154143983](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007154143983.png)



File Installation Key 창에서는 라이선스 활성화시 안내받은 숫자와 하이픈 (`-`)만으로 구성된 값을 입력하고, `Next` 버튼을 누릅니다.

![image-20211007154302331](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007154302331.png)



설치 타입은 `Typical`을 선택하고, `Next`버튼을 누릅니다.

![image-20211007154336147](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007154336147.png)

설치할 폴더를 선택하고 `Next`버튼을 누릅니다.

![image-20211007154356122](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007154356122.png)

설치가 완료되면, 다음과같이 확장자를 연결하는지 묻는 창이 나오는데, `.fig`, `.m`, `.mat`과 같은 확장자들은 거의 MATLAB에서만 사용되므로, 여기에서는 `Yes to All`을 선택합니다.

![image-20211007154507215](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007154507215.png)



[v] Activate MATLAB를 선택하고, `Next`버튼을 누릅니다.

![image-20211007154602050](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007154602050.png)



이미 Activation file을 받았으므로, 인터넷없이 수동으로 활성화하도록 선택하고,  `Next`버튼을 누릅니다.



![image-20211007154643199](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007154643199.png)



Offline Activation창에서는, 우측의 `Browse`버튼을 눌러, 이미 받은 `license.lic`파일을 찾아서 선택합니다.

![image-20211007154926796](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007154926796.png)



Finish 버튼을 누르면, MATLAB이 실행한다.

![image-20211007154944123](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007154944123.png)



![image-20211007155043605](D:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-07-컴퓨터-매트랩 activation.assets\image-20211007155043605.png)

# 참고



### offline activation방법 

https://www.mathworks.com/matlabcentral/answers/259627-how-do-i-activate-matlab-or-other-mathworks-products-without-an-internet-connection



