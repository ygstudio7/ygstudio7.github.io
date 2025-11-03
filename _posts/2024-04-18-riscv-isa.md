

RISC-V has standardized a series of **standard extensions** beyond the integer base instructions which can be implemented or omitted as desired depending on the design goals (e.g. energy/area/performance/storage goals).

By default, only the [core ISA](https://en.wikichip.org/w/index.php?title=risc-v/integer_base&action=edit&redlink=1) must be implemented presenting great opportunity for area and energy optimization. However, additional functionality is sometimes desired. RISC-V comes with a series of standard extensions that enable additional functionality beyond the [core ISA](https://en.wikichip.org/w/index.php?title=risc-v/integer_base&action=edit&redlink=1) such as [floating point](https://en.wikichip.org/w/index.php?title=floating_point&action=edit&redlink=1) and operations and [bit](https://en.wikichip.org/wiki/bit) [manipulation](https://en.wikichip.org/w/index.php?title=bit_manipulation&action=edit&redlink=1). Extensions can be implemented and omitted as desired. Those extensions are:



# Base Instructions

|   Name    |                         Description                          | Version | Status | Instruction Count |
| :-------: | :----------------------------------------------------------: | :-----: | :----: | :---------------: |
| **RV32I** |            Base Integer Instruction Set - 32-bit             |   2.1   | Frozen |        49         |
|   RV32E   | Base Integer Instruction Set (embedded) - 32-bit, 16 registers |   1.9   |  Open  |   Same as RV32I   |
| **RV64I** |            Base Integer Instruction Set - 64-bit             |   2.1   | Frozen |        14         |
|  RV128I   |            Base Integer Instruction Set - 128-bit            |   1.7   |  Open  |        14         |

# Standard Instruction Extensions

| Extension |                                                              |      |        |               |
| :-------: | ------------------------------------------------------------ | ---- | ------ | ------------- |
|   **M**   | Standard Extension for **Integer Multiplication and Division** | 2.0  | Frozen | 8             |
|   **A**   | Standard Extension for **Atomic Instructions**               | 2.1  | Frozen | 11            |
|   **F**   | Standard Extension for **Single-Precision Floating-Point**   | 2.2  | Frozen | 25            |
|   **D**   | Standard Extension for **Double-Precision Floating-Point**   | 2.2  | Frozen | 25            |
|   **G**   | Shorthand for the base and above extensions                  | n/a  | n/a    | n/a           |
|     Q     | Standard Extension for Quad-Precision Floating-Point         | 2.2  | Frozen | 27            |
|     L     | Standard Extension for Decimal Floating-Point                | 0.0  | Open   | Undefined Yet |
|   **C**   | Standard Extension for **Compressed Instructions**           | 2.0  | Frozen | 36            |
|   **B**   | Standard Extension for **Bit Manipulation**                  | 0.90 | Open   | 42            |
|     J     | Standard Extension for Dynamically Translated Languages      | 0.0  | Open   | Undefined Yet |
|     T     | Standard Extension for Transactional Memory                  | 0.0  | Open   | Undefined Yet |
|   **P**   | Standard Extension for **Packed-SIMD Instructions**          | 0.9  | Open   | 325           |
|   **V**   | Standard Extension for **Vector Operations**                 | 0.7  | Open   | 186           |
|     N     | Standard Extension for User-Level Interrupts                 | 1.1  | Open   | 3             |
|     H     | Standard Extension for Hypervisor                            | 1.0  | Frozen | 2             |
|     S     | Standard Extension for Supervisor-level Instructions         | 1.12 | Open   | 7             |





# Vetor Extensions

https://github.com/riscv/riscv-v-spec/blob/master/example/vvaddint32.s

```
    .text
    .balign 4
    .global vvaddint32
    # vector-vector add routine of 32-bit integers
    # void vvaddint32(size_t n, const int*x, const int*y, int*z)
    # { for (size_t i=0; i<n; i++) { z[i]=x[i]+y[i]; } }
    #
    # a0 = n, a1 = x, a2 = y, a3 = z
    # Non-vector instructions are indented
vvaddint32:
    vsetvli t0, a0, e32, ta, ma  # Set vector length based on 32-bit vectors
    vle32.v v0, (a1)         # Get first vector
      sub a0, a0, t0         # Decrement number done
      slli t0, t0, 2         # Multiply number done by 4 bytes
      add a1, a1, t0         # Bump pointer
    vle32.v v1, (a2)         # Get second vector
      add a2, a2, t0         # Bump pointer
    vadd.vv v2, v0, v1       # Sum vectors
    vse32.v v2, (a3)         # Store result
      add a3, a3, t0         # Bump pointer
      bnez a0, vvaddint32    # Loop back
      ret                    # Finished
```



# Toolchain support

참고: https://www.reddit.com/r/RISCV/comments/13iv9xa/use_simd_from_c/?rdt=49651

P extension is not ratified yet and is not even frozen yet. 

Some manufacturer claims "P" in the ISA, but it means draft version.

We we use a core from Andes technology, it means Andes DSP/SIMD extension.

**We need to check if Andes toolchain supports SIMD or Vector instructions automatically generated from plain/pure C code.**

As far as I know, Andes support intrinsic functions in their compiler, but user needs to manually insert SIMD/Vector functions in C code.

It means that compiler support P extension, but the op codes not generated from plain c code.



P 확장은 아직 비준되지 않았으며 동결되지도 않았습니다. 현재 계획은 올해 4분기에 동결하는 것이다. 현재 버전은 "0.9.11-draft-20211209"이며 그 이후로 7개의 PR/병합이 추가되었습니다.

ISA 문자열에서 "P"를 주장하는 제조업체는 일부 초안 버전을 의미할 수 있으며, Andes의 코어를 사용하는 경우 독점 NDS32 ISA에서 RISC_V 코어로 직접 복사한 Andes DSP/SIMD 확장을 의미할 수 있습니다.



P는 아직 비준되지 않았습니다. 이론적으로 컴파일러는 P를 사용하도록 Intrinsics를 정의할 수 있지만 아마도 그렇지 않을 것입니다.

결국 SIMD 명령어를 배우고 어셈블러를 사용하든 C를 사용하든 최적의 방식으로(충분히 가까운) SIMD 명령어를 결합하는 방법을 알아내야 합니다.



내장 함수가 컴파일러에 정의되어 있는지 어떻게 확인할 수 있나요?



# 참고: 

https://en.wikichip.org/wiki/risc-v/standard_extensions

https://riscv.org/technical/specifications/

https://dl.acm.org/doi/10.1145/3569939#:~:text=There%20are%20325%20instructions%20defined,bit%20operands%20for%20arithmetic%20instructions.

: P has 325 SIMD instructions



https://github.com/riscv/riscv-v-spec/blob/master/v-spec.adoc

: Vector extension

https://five-embeddev.com/riscv-user-isa-manual/Priv-v1.12/p.html

