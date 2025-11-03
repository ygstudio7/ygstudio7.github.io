참고: https://ttend.tistory.com/712

# vmware 디스크 용량 늘리기, 디스크 추가하기



# 가상머신 디스크 용량 늘리기

vmware로 가상머신을 사용하다가 디스크 용량이 부족할 경우, vmare 설정에서 디스크 크기를 늘리는 메뉴를 사용할 수 있다. 

가상머신을 종류한 상태에서,

`VM` -> `Settings` 를 선택한 후, `VIrtual Machine Settings`창에서 `Hardware`옵션의 `Hard Disk`를 선택한다. 

우측의 `Expand`버튼을 선택하면 디스크 크기를 조정할 수 있다.

![image-20210515132749341](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210515132749341.png)





# snapshot에 디스크 추가



가상머신의 snaptshot을 찍은 후에는 위 `Expand`메뉴는 비활성화되어 사용할 수 없다.

이경우에는 별도의 하드 디스크를 추가해서 사용할 수 있다.

동일한 `Virtual Machine Settings`에서 하단의 `Add` 버튼을 선택한다.

![image-20210515133134245](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210515133134245.png)



`Add Hardware Wizard` 창이 나타나면, `Hard Disk`를 선택하고, `Next` 버튼을 누른다.

![image-20210515133344134](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210515133344134.png)



`Virtual disk type`은 `SCSI`로 설정하고, `Next`버튼을 선택한다.

![image-20210515133417866](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210515133417866-1621110859456.png)



`Create a new virtual disk`를 선택하고, `Next`버튼을 누른다.

![image-20210515153524737](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210515153524737.png)



디스크 용량을 선택하고, `Next`버튼을 누른다.

![image-20210515153557874](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210515153557874-1621118158216.png)



Disk file이름을 입력하고, `Finish`를 선택한다.

![image-20210515153614311](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210515153614311.png)

기존의 20GB 하드 디스크이외에 20GB의 하드 디스크가 추가되었다.

![image-20210515153710942](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210515153710942-1621118231542.png)



Computer management에 들어가서 해당 디스크를 추가하고 포맷하면 된다.

![image-20210515154519698](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210515154519698.png)

![image-20210515154543861](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210515154543861.png)