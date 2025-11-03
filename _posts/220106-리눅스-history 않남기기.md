https://neo-blog.tistory.com/22

여기는 ~/.bash_history의 내용이고,

![image-20220106111528046](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2022-01-06-리눅스-history 않남기기.assets\image-20220106111528046.png)

아래는 HISTFILE에 남아 있는 내용이다.

![image-20220106111848831](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2022-01-06-리눅스-history 않남기기.assets\image-20220106111848831.png)



terminal에서 사용후에는 아래 명령을 실행하고 나면, HISTFILE이 가리키는 .bash_history 가 없어지므로, 현재 터미널에서 사용한 명령들은 ~/.bash_history에 들어가지 않는다.

현재 세션에서 사용한 명령들은 남지 않게 된다.

```bash
$ echo $HISTFILE
/home/ygkim/.bash_history
$ unset HISTFILE
```



따라서, .bash_history를 수정후, unset HISTFILE을 실행하면, 현재 세션의 기록이 없어지므로, 원하는 기록들을 삭제할 수 있다.

삭제하고자 하는 기록들.

```
cp /home/mjsung/EN675_SINGLE/HW/RTL_SIM ./HW/
cp -rf /home/mjsung/EN675_SINGLE/HW/RTL_SIM ./HW/
sudo cp -rf /home/mjsung/EN675_SINGLE/HW/RTL_SIM ./HW/
su cp -rf /home/mjsung/EN675_SINGLE/HW/RTL_SIM ./HW/
cp -rf /home/mjsung/EN675_SINGLE/HW/RTL_SIM ./HW/
ll
mkdir HW
chown mjsung:mjsung HW
cp -rf /home/mjsung/EN675_SINGLE/HW/RTL_SIM ./HW/
cd SIM
cp -rf /home/mjsung/EN675_SINGLE/HW/SIM/LIBRARY/ ./HW/SIM/
cp -rf /home/mjsung/EN675_SINGLE/HW/SIM/LIBRARY ./HW/SIM/
cd ..
ll
cd HW
ll
chown ygkim:ygkim RTL_SIM
cd RTL_SIM
ll
chown ygkim:ygkim main_asic.v 
gedit ~/.bash_history 
history
exit
history
exit
```

