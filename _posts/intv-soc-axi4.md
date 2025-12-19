History

- 251201: initial



# Part 1: Basic

AXI4는 ARM의 AMBA(Advanced Microcontroller Bus Architecture) 규격 중 가장 널리 쓰이는 고성능 SoC 인터커넥트 규격입니다. CPU–DMA–Accelerator–Memory 같은 다양한 IP 사이를 **고속·병렬·비차단 방식**으로 연결하는 데 최적화되어 있습니다.
 아래에서 **구조 → 5개 채널 → Handshake → Burst → 주요 신호 → 장점** 순으로 깔끔하게 정리해드립니다.

------

# ⭐ AXI4 개요

- ARM AMBA 4 세대 버스 프로토콜
- 완전 파이프라인 구조 (Address, Data 분리)
- **5개의 독립 채널** 사용
- Out-of-order, burst, multiple outstanding transaction 가능
- 고대역폭 메모리 액세스(DDR Controller 등)에서 사실상 표준

------

# 🔶 AXI4의 5개 채널

AXI4는 Read/Write 각각 주소와 데이터를 **분리된 채널로 전송**합니다.

## 1) **AW (Write Address Channel)**

- Write transaction의 시작(주소/길이/타입 등 정보)
- 주요 신호:
   `AWADDR`, `AWLEN`, `AWSIZE`, `AWBURST`, `AWID`, `AWPROT`, `AWVALID/READY`

## 2) **W (Write Data Channel)**

- 실제 Write 데이터 전송
- 데이터는 Beat 단위로 전송
- 주요 신호:
   `WDATA`, `WSTRB`, `WLAST`, `WVALID/READY`

## 3) **B (Write Response Channel)**

- Write 완료 신호(OKAY/SLVERR/DECERR 등)
- 주요 신호:
   `BID`, `BRESP`, `BVALID/READY`

## 4) **AR (Read Address Channel)**

- Read transaction 주소 및 속성
- 주요 신호:
   `ARADDR`, `ARLEN`, `ARSIZE`, `ARBURST`, `ARID`, `ARVALID/READY`

## 5) **R (Read Data Channel)**

- Read 데이터 반환
- Burst 구조로 여러 beat
- 주요 신호:
   `RDATA`, `RRESP`, `RVALID/READY`, `RLAST`, `RID`

------

# 🔶 AXI4 Handshake (VALID/READY)

AXI 모든 채널은 동일한 핸드셰이크:

```
if VALID && READY -> 전송 성공 (1 beat)
```

- Master: VALID
- Slave: READY
- 둘 다 1이 되는 클럭 싸이클에 전송 확정
- 파이프라이닝이 자유롭고, 슬레이브 백프레셔(back-pressure) 가능

------

# 🔶 Burst Transfer 방식

AXI4는 1회 주소 전송으로 여러 Data beat 전송 가능.

| 타입      | 설명                                  |
| --------- | ------------------------------------- |
| **FIXED** | 주소 고정 (FIFO, peripheral 등)       |
| **INCR**  | 증가 주소 burst (DRAM 일반적)         |
| **WRAP**  | wrap-around burst (cache line refill) |

### Burst 관련 필드

- **LEN**: beat 개수 - 1
- **SIZE**: beat당 바이트 수(=log2(bytes))
- **BURST**: type(FIXED/INCR/WRAP)

예) 16byte Burst (4beat × 4B)

```
AWLEN = 3   (4 beat)
AWSIZE = 2  (4 bytes per beat)
AWBURST = INCR
```

------

# 🔶 AXI4 Full vs AXI4-Lite vs AXI4-Stream

| 규격           | 특징                         | 용도                          |
| -------------- | ---------------------------- | ----------------------------- |
| **AXI4**       | Burst + ID + high throughput | CPU–DDR, DMA, Accelerator     |
| **AXI4-Lite**  | Simple, no burst, no ID      | Register access (MMIO)        |
| **AXI-Stream** | 주소 없음, Pure data stream  | Video/ISP/ML/Network datapath |

------

# 🔶 AXI4 주요 특징 요약

- 완전 파이프라이닝
- Read/Write 채널 독립 → 병렬 처리
- 여러 outstanding transaction (ID 기반)
- Back-pressure 지원
- High bandwidth & low latency 구조
- Cache/DDR와 최적화된 bus



# Part 2: 

