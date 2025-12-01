# 계산방법

# 예1: EN675

참고:

https://www.notion.so/RISC-V-Renewal-83c20866da854f60a9d2e6249a55b824#b769a12eca0b4e88910c28386d7edacf

spec: RISC-V, 4 tiles, L1 16kB, L2 512kB

clock: 675, 337.5MHz

samsung 28nm LP process,

9 track cell, 1st pass RVT, 2nd pass RVT+LVT

```
Area : 600만 gate count
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5ed1ad04-60c9-4032-b59e-2bf842db7476/Untitled.png)

![image-20251110233410098](assets/image-20251110233410098.png)

LVT cell : 대략 22만개 ⇒ 기존 CPU를 594MHz 주파수로 합성했을 때에는 약 10만개 정도였음

**QOR (Quality of Result) : 메모리 path에서 WNS가 -0.28ns 정도 발생, iCLK_TILE에서는 -0.09ns WNS**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d5cf8560-feb9-46d9-aa6a-a87adb3c8e75/Untitled.png)

![image-20251110233423888](assets/image-20251110233423888.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a3b21b14-5bee-45c7-a528-42a2806820a2/Untitled.png)

![image-20251110233433913](assets/image-20251110233433913.png)

Formality EQ check : SUCCESS

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d9dde9df-b434-41a7-a76c-885ec151d9bd/Untitled.png)

![image-20251110233440437](assets/image-20251110233440437.png)