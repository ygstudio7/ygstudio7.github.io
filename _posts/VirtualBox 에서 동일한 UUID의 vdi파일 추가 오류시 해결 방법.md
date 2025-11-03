### VirtualBox 에서 동일한 UUID의 vdi파일 추가 오류시 해결 방법

참고: https://stackoverflow.com/questions/44114854/virtualbox-cannot-register-the-hard-disk-already-exists/45391121

VirtualBox Cannot register the hard disk already exists

c:\Program Files\Oracle\VirtualBox\ 폴더로 이동후에,

```
VBoxManage.exe internalcommands sethduuid x:\Project\vbox\WinLab10.vdi
UUID changed to: d4e668ef-d85b-4657-b5e9-feb105cff11b
```
