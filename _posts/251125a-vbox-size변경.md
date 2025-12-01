

## 크기 확인시

VBoxManage showhdinfo "D:\xvbox\WinLab10_211026b.vdi"

```
UUID:           8fb7a549-75ce-4e7f-9ed1-139d7e9ac779
Parent UUID:    base
State:          created
Type:           normal (base)
Location:       D:\xvbox\WinLab10_211026b.vdi
Storage format: VDI
Format variant: dynamic default
Capacity:       60620 MBytes
Size on disk:   60187 MBytes
Encryption:     disabled
Property:       AllocationBlockSize=1048576
In use by VMs:  WinLab10b (UUID: f8a0294f-bdd5-4450-b1af-ceb442526089)
```



## 변경시

VBoxManage modifyhd "D:\xvbox\WinLab10_211026b.vdi" --resize 80000



## **C 드라이브 확장 (Extend Volume)**

Disk Management에서:

1. **C: 파티션 우클릭**
2. **Extend Volume** 클릭
3. Next → Next → Finish

