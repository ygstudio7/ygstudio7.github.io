



## 자동 실행없이 설치파일 실행



```dos
Opera_76.0.4017.94_Setup_x64.exe --launchopera=0
```



![image-20210823115927715](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210823115927715.png)



## 환경 변수 추가

![image-20210823115755301](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210823115755301.png)



opera 브라우저 실행후 > 좌측 opera 아이콘 선택 > `도움말` 선택 > `Opera 정보`선택하면, 업데이트 확인을 사용하지 않는다고 나온다.



![image-20210823120033985](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210823120033985.png)



![image-20210823120105801](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210823120105801.png)





I therefore think the following solution makes more sense:

- **Run the installer with `--launchopera=0`**
- **Start Opera with `--disable-update`**
- 

# **R**eference

https://www.reddit.com/r/operabrowser/wiki/opera/rollback_version

