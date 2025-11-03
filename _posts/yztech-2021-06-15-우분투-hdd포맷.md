

# 우분투 하드 드라이브 포맷하는 방법

우분투에서 disks 프로그램을 사용하면, 하드 드라이브를 포맷할 수 있다. 

disks 프로그램을 연다. 대시를 열어서 `disks`를 입력하면 빠르게 찾을 수 있다. 

![Screenshot from 2021-06-15 09-48-24](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-06-15-우분투-hdd포맷.assets\Screenshot from 2021-06-15 09-48-24.png)



좌측에서 포맷하고 싶은 드라이브를 선택한다. 드라이브 포맷을 하면 파티션의 모든 부분이 지워지끼 때문에 포맷할 드라이브를 신중히 선택한다. 

기어 버튼을 누르고 `Format Partition` 을 선택하면, 파일 시스템을 구성하는 새 창이 열린다.

![image-20210615105624313](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-06-15-우분투-hdd포맷.assets\image-20210615105624313.png)

![image-20210615105644176](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-06-15-우분투-hdd포맷.assets\image-20210615105644176.png)



`Volume Name`에 원하는 드라이브 이름을 지정하면, 장치 식별이 편리하다.

`Erase`는 기본적으로 `OFF`로 한다. 느리지만 모든 데이터를 삭제하고 안전하게 포맷하고자할 경우에는 `ON`을 선택한다.

`Type`에는 사용하고자 하는 파일 시스템을 선택한다. 

- Ext4: 리눅스에서만 사용할 경우 선택
- NTFS: 윈도우에서만 사용할 경우 선택
- FAT: 리눅스, 윈도우, Mac 모두에서 사용할 경우 선택
- 

![Screenshot from 2021-06-15 09-50-44](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-06-15-우분투-hdd포맷.assets\Screenshot from 2021-06-15 09-50-44.png)



도문 데이터가 지워진다는 경고 문구와 함께 우측 상단에 `Format` 실행 버튼을 누른다.

![Screenshot from 2021-06-15 09-51-03](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-06-15-우분투-hdd포맷.assets\Screenshot from 2021-06-15 09-51-03.png)



포맷후에는 원하는 위치에 디스크 마운트해서 사용하면 된다.



#참고:

https://ko.wikihow.com/%EC%9A%B0%EB%B6%84%ED%88%AC%EB%A1%9C-%ED%95%98%EB%93%9C-%EB%93%9C%EB%9D%BC%EC%9D%B4%EB%B8%8C-%ED%8F%AC%EB%A7%B7%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95

https://www.manualfactory.net/10607