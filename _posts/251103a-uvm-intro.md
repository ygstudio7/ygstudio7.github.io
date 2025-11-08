# [] 소개

UVM은 디지털 설계와 SoC 검증을 위한 표준적인 방법으로, systemverilog로 구현되었다. 

modular, 재구성가능한 tb 요소돌이 생성해서 검증 과정을 도와준다

tb개발, 시뮬레이션 및 분석 방법에 대한 예제나 가이드도 포함한다.



# [] 구성요소

사용자가 재사용할 수 있도록 이미 정의된 클래스와 메소드를 포함

## TB 요소들

tb 요소들을 만들 수 있도록 기본 클래스들을 제공: driver, monitor, scoreboard and agent

아래는 이미 만들어진 uvm요소들 (`*.uvm`)을 확장해서 만들어진 전형적인 검증 환경



    [seq] -> [drv] -> [dut] -> [mon] -> [sb]
               v                         ^  
               ---------------------------

![](https://www.chipverify.com/images/uvm/uvm-tb.gif)



알아야 할 것들

* transactions: 

* phases: creation initialiation, execution

* messaging/reporting: warning, errors, debug 정보

* configuration: config db가 있어서 사용자가 config 정보를 저장하고, 읽어낼 수 있다

* functional cov: design이 제대로 test되었는지 확인하는 coverage를 추적할 수 있는 방법 제공

* reg abs layer (RAL): register map을 생성하고, 접근하는 쉬운 방법 제공



# [ ] 준비사항

* sv로 구현되었으므로, sv syntax를 알아야 하고

* class, inheritance, randomization을 알아야 함

* incisive, quesa. vcs 등 시뮬레이션 툴





# [ ] 설치



skip



# [] uvm

- UVM은 SystemVerilog 클래스 프레임워크이며, 기능적 검증(Testbench)을 구축하기 위한 일련의 기반 클래스들을 제공합니다. [ChipVerify](https://www.chipverify.com/uvm/uvm-introduction)

- 즉, SystemVerilog가 언어적 기초라면 UVM은 그 위에 쌓이는 “검증 방법론/프레임워크”입니다. [ChipVerify](https://www.chipverify.com/uvm/uvm-introduction)

- 

# [] 왜  UVM 필요?

(-) 개발자 마다 서로 다른 방식으로 warning, error, debug 메시지를 출력함

(+) uvm은 표준화된 reporting 방법을 제공

(+) 엔지니어는 검증 platform구축이 아닌 설계 검증에만 집중 가능: tb structure는 구축할 필요없이 stimulus에만 신경쓰면 된다



- 검증(Testbench) 구축 시 팀 간, IP 재사용 시, 여러 프로젝트 간 일관된 방식이 없으면 유지보수 및 통합이 어렵습니다. UVM은 이러한 ‘검증환경의 표준화’와 ‘재사용성’을 제공합니다. [ChipVerify](https://www.chipverify.com/uvm/uvm-introduction)

- 예컨대 경고/에러/디버그 메시지 처리, 드라이버-시퀀서 핸드쉐이크 등이 프레임워크에 의해 제공되므로 엔지니어는 본질적인 “검증” 과제에 집중할 수 있습니다.



# [] 어떻게 동작

- 검증환경에는 일반적으로 드라이버(driver), 모니터(monitor), 자극 생성기(stimulus generator), 스코어보드(scoreboard) 등이 필요합니다. UVM은 이러한 주요 컴포넌트에 대한 기반 클래스(base class)를 제공하고, 인스턴스화, 연결(connection), 구성이 표준화되어 있습니다. [ChipVerify](https://www.chipverify.com/uvm/uvm-introduction)

- 즉, 테스트벤치의 골격(scaffolding)을 UVM이 제공해 주므로, 사용자(검증 엔지니어)는 프로토콜 특화(stimulus, sequence)를 구현하는 데 더 집중할 수 있습니다.

# [] uvm 클래스 계층

[cg]

- (user-defined classes)를 구현합니다. 예컨대 AXI 프로토콜용 드라이버를 만든다면 `uvm_driver`를 상속받아 구현할 수 있습니다. [ChipVerify](https://www.chipverify.com/uvm/uvm-introduction)

- 클래스 구조는 크게 두 가지 브랜치(branch)로 나뉩니다:
  
  - `uvm_component` 아래: 드라이버, 모니터, 에이전트(agent)와 같은 검증컴포넌트들
  
  - `uvm_transaction` (혹은 `uvm_sequence_item`) 아래: 트랜잭션(데이터 오브젝트)들 [ChipVerify](https://www.chipverify.com/uvm/uvm-introduction)

[나]

복잡한 class는 기본 클래스를 확장해서 생성 가능

예. AXI 를 지원하는 새로운 driver class는 uvm_driver를 확장해서 생성 가능

protocol을 위한 stimulus는 uvm_sequence_item을 확장해서 만들 수 있다

uvm_component: driver, monitor, agents와 같이 검증용 components를 정의하는 클래스

uvm_transaction: 위 클래스에서 사용되는 data 객체들을 정의하는 클래스들



![UVM Class Hierarchy](https://www.chipverify.com/images/uvm/uml_uvm_class_hier.svg)

 

# 주요 클래스들

## uvm objects

복사(copy), 비교(compare), 출력(print) 등의 공통 기능을 갖는 객체 기반 클래스입니다. 예컨대 검증환경의 설정(configuration) 클래스를 `uvm_object`로부터 상속하여 구현합니다.



## uvm sequence

stimulus  생성용

자극(stimulus)을 담는 컨테이너 역할을 하며, 여러 시퀀스(sequence)를 조합하거나 다른 시퀀스 안에 포함시켜 다양한 시나리오를 구현할 수 있습니다.

## uvm sequence items

시퀀스에 의해 생성되어 드라이버에 의해 DUT로 보내질 데이터 객체(transaction)들입니다. `uvm_sequence_item`을 상속하여 사용합니다.

예: apb trnasactin위해서는 address, write data, read data, access type등의 저으이되어야 한다



## uvm components

모든 tb 에 필요한 요소들

검증환경의 구조적 요소(드라이버, 모니터, 스코어보드 등)로서 `uvm_component` 기반으로 구현됩니다. 표로 정리하면:

- `uvm_driver`: DUT에 신호를 구동

- `uvm_monitor`: DUT의 출력/포트 신호를 관찰

- `uvm_sequencer`: 자극 패턴 생성

- `uvm_agent`: Sequencer + Driver + Monitor 포함

- `uvm_env`: 전체 검증 환경을 포함

- `uvm_scoreboard`: 테스트 결과 판정(Pass/Fail) [C](https://www.chipverify.com/uvm/uvm-introduction)

## register layer

register제어를 쉽게 해주는 역할

설계 내 제어레지스터들을 검증하기 위한 모델(register model)을 UVM이 제공합니다. 이전에는 각 프로젝트마다 별도 구현해야 했던 레지스터 검증 로직을 보다 쉽게 구성할 수 있습니다.

## TLM connections

트랜잭션을 컴포넌트 간에 전달하는 인터페이스로, UVM은 TLM Port/Export, Blocking/Non-blocking, FIFO, 분석포트(analysis port) 등 다양한 통신 모델을 지원합니다.

## uvm phases

모든 components사이의 sync 맞추기

검증환경 내 컴포넌트들이 동기화(synchronization)되어 실행되는 단계(phases)를 정의합니다. 예컨대 build 단계, connect 단계, run 단계, final 단계 등이 있습니다.



### Summary / 요약

- `uvm_object` → Testbench 설정 및 데이터 객체

- `uvm_component` → 드라이버/모니터/스코어보드 등 검증 컴포넌트

- `uvm_sequence_item` → 트랜잭션 객체

- `uvm_sequence` → 자극 패턴을 정의하는 시퀀스 클래스 [ChipVerify](https://www.chipverify.com/uvm/uvm-introduction)


