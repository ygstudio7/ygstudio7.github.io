## Keyword

AHB to APB bridge, no wait state,

2022년 12월 13일

2022년 12월 15일

https://testbench4u.com/2018/09/08/apb-protocol-interview-questions/

1. How should AHB to APB bridges handle accesses that are not 32 bits?

   APB 가 32 bits이하일때는 AHB의 32 bits의 적절한 bits에 연결해야 한다.

   : When AHB transfers to an APB slave with less than 32 bits, the slave should be located on the appropriate bits of the AHB data bus.

2. What is there no wait signal on the APB?

   APB는 I/F를 가능한 간단하게 구현하기 위해 설계되었다.

   따라서, single access만 지원하는 느린 devices들에 사용하는데, UART, control registers들이다.

   이들은 보통 다음과 같이 사용한다.

   먼저 status register를 확인해서, data가 있는지 결정하고, data register를 접근한다.

   이러한 방식은 wait state없이 가능하므로, APB에 쉽게 연결할 수 있다.

   APB was designed to implement as simple as possible.

   This simple interface makes it easier to connect APB peripherals.

   Many APB peripherals are slow devices such as UART or control registers.

   Typically slow devices provides status registers to check the device status.

   Thus, it first access a status register to determine further access is required.

   And then it accesses the data or other registers.

   Thus, it does not require wait states.

# Signal Description

[5.15.7. APB Interface Signal Types](https://www.intel.com/content/www/us/en/docs/programmable/683609/21-3/interface-signal-types.html)

# 참고

[On-Chip-Bus-Interconnections.pdf](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ac4fe57f-ff42-4e4b-9563-cee83769b1af/On-Chip-Bus-Interconnections.pdf)

Q/A