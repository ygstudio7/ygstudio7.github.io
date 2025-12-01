# FF와 latch 차이



|            | F/F               | latch           |
| ---------- | ----------------- | --------------- |
|            | edge sensitive    | level sensitive |
| glitch     | (+) not sensitive |                 |
| # of gates |                   | (+) less        |
| speed      |                   | (+) faster      |

## 발생 조건

A conditional statement does not cover all possible cases.

: in the `if` or `case` statements, there is no `else` or `default` statements

: 즉 모든 조건에서 출력이 정해지지 않으면, 남은 신호는 latch로 생성된다.

When the output is not assigned in all the conditions, a latch gets inferred on the left cases.

# 참고

https://www.myhdl.org/docs/examples/flipflops.html

: verilog code 존재

https://vhdlwhiz.com/why-latches-are-bad/

: 자세한 설명

https://inst.eecs.berkeley.edu/~eecs151/sp19/files/discussion3.pdf

:

https://nandland.com/how-to-avoid-creating-a-latch/#:~:text=To avoid this%2C make sure,not include the default assignment

:

# Lab

[Simulating D Flip-Flop on Xilinx: ISE Design Suite| Verilog HDL| Behavioral Modeling| Digital Design](https://www.youtube.com/watch?v=m8uZjAMPmC4)

https://xilinx.github.io/xup_fpga_vivado_flow/