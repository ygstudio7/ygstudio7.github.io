[EN675_Intro_RISC-V_200618.pdf](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a3f0f56f-7de9-442b-b688-33a053a7a1fb/EN675_Intro_RISC-V_200618.pdf)

: EN675에 적용된 64bit RISC-V ㅇ와 ARM CPU 비교

: 4번째 page: EN675 CPU performance benchmark (Dhrystone, coremark)

| Performance                    | ARM Coretex-A5 | RISC-V Rocket (U54) | EN675 RISC-V |                        |
| ------------------------------ | -------------- | ------------------- | ------------ | ---------------------- |
| Dhrystone benchmark: DMIPS/MHz | 1.57           | 1.72                | 1.80         | for general processors |
| Coremark/MHz                   |                | 2.75                | 2.95         | for embeded processors |
|                                |                |                     |              |                        |

EN675용 CPU

- RV64 IMAFDC ISA를 지원:
  - Integer, Multipler, Atomic operation, single-point Floating point, Double-precision Floating point, Compressed instruction

[ISA](https://www.notion.so/ISA-81f78e09b0324363aab3d5e7b44c43ce?pvs=21)