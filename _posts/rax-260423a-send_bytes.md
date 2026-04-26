
```
   **task send_bytes(input logic [7:0] data[$], input bit close_chn = 1);

  

      int cb, cbit;  // cur byte, cur bit

      int ncks;

  

      ncks = data.size() * 8 / (2 - sdr) / channel_width;

      cbit = 7;

  

      check_sip_HiZ: assert (!check_bus_free | vif_ls_io.lspi_din===4'bzzzz)

        else $display ("SPI but not HiZ: %b", vif_ls_io.lspi_din);

  

      vif_ls_io.pad_lspi_sck = cpol;

      vif_ls_io.pad_lspi_csn = 0;

      vif_ls_io.en_spiclk = 1;

      @(vif_ls_io.ev_spiclk_q);

  

      // FIXME: DDR?

  

      set_drv_data(data[cb], cbit);

      cbit -= channel_width;

      if(!cpha | !sdr) begin

        @(vif_ls_io.ev_spiclk_q);

        @(vif_ls_io.ev_spiclk_q);

      end

  

      repeat (ncks) begin

  

        @(vif_ls_io.ev_spiclk_q);

        vif_ls_io.pad_lspi_sck = !vif_ls_io.pad_lspi_sck;

        @(vif_ls_io.ev_spiclk_q);

  

        if(!cpha | !sdr)          // after pos edge

          if(cb < data.size()) begin

            set_drv_data(data[cb], cbit);

            cbit -= channel_width;

          end

  

        if (cbit < 0) begin

          cbit = 7;

          cb += 1;

        end

  

        @(vif_ls_io.ev_spiclk_q);

        vif_ls_io.pad_lspi_sck = !vif_ls_io.pad_lspi_sck;

        @(vif_ls_io.ev_spiclk_q);

  

        if(cpha | !sdr) begin           // after neg edge

          set_drv_data(data[cb], cbit);

          cbit -= channel_width;

        end

      end

  

      if (close_chn) begin

        set_drv_data(0, -1);

        vif_ls_io.pad_lspi_csn = 1;

        repeat(min_csn_high) @(vif_ls_io.ev_spiclk_q);

        vif_ls_io.en_spiclk = 0;

      end

  

    endtask**
```

아래 task는 **SPI byte들을 bit 단위로 drive하면서 SCK를 toggle하는 코드**입니다.

```verilog
task send_bytes(input logic [7:0] data[$], input bit close_chn = 1);
```

`data[$]`는 보낼 byte queue입니다. `close_chn`이 1이면 전송 후 CSN을 올리고 clock enable을 끕니다.

```verilog
int cb, cbit;  
int ncks;
```

`cb`는 current byte index, `cbit`는 current bit position입니다.  
`ncks`는 clock cycle 반복 횟수입니다.

```verilog
ncks = data.size() * 8 / (2 - sdr) / channel_width;
```
보낼 총 bit 수를 기준으로 필요한 SCK cycle 수를 계산합니다.  
`channel_width`가 1이면 1bit씩, 4이면 nibble씩 전송하는 구조로 보입니다.

```verilog
cbit = 7;
```

`MSB`부터 보내기 위해 bit index를 7로 시작합니다.

주의: 여기서 **`cb = 0;` 초기화가 빠져 있습니다.**

```verilog
check_sip_HiZ: assert (!check_bus_free | vif_ls_io.lspi_din===4'bzzzz)
else $display ("SPI but not HiZ: %b", vif_ls_io.lspi_din);

```
bus가 free 상태인지 확인합니다. `check_bus_free`가 true이면 `lspi_din`이 Hi-Z인지 검사합니다. Hi-Z가 아니면 메시지를 출력합니다.

```verilog
vif_ls_io.pad_lspi_sck = cpol;
```
SPI clock을 idle polarity로 둡니다.  
CPOL=0이면 idle low, CPOL=1이면 idle high입니다.

```verilog
vif_ls_io.pad_lspi_csn = 0;
```
CSN을 low로 내려서 SPI transaction을 시작합니다.

```verilog
vif_ls_io.en_spiclk = 1;
```

SPI clock 동작을 enable합니다.

```verilog
@(vif_ls_io.ev_spiclk_q);
```

내부 clock event 한 번 기다립니다.

// FIXME: DDR?

DDR 동작은 아직 완전히 정리되지 않았다는 표시입니다.

```verilog
set_drv_data(data[cb], cbit);
```

첫 data bit를 drive합니다.  
CPHA=0에서는 첫 sampling edge 전에 data가 valid해야 하므로 이 pre-drive가 필요합니다.  
단, `cb` 초기화가 없어서 위험합니다.

```verilog
cbit -= channel_width;
```

다음에 보낼 bit 위치로 이동합니다.

if(!cpha | !sdr) begin

CPHA=0이거나 SDR이 아니면 추가 delay를 줍니다.  
여기는 `|`보다 `||`가 적절합니다.

@(vif_ls_io.ev_spiclk_q);  
@(vif_ls_io.ev_spiclk_q);

clock event 두 번 기다립니다.

repeat (ncks) begin

계산된 clock cycle 수만큼 반복합니다.

@(vif_ls_io.ev_spiclk_q);

edge 만들기 전 event를 기다립니다.

vif_ls_io.pad_lspi_sck = !vif_ls_io.pad_lspi_sck;

SCK를 toggle합니다.  
CPOL=0이면 첫 toggle은 posedge, CPOL=1이면 첫 toggle은 negedge입니다. 즉 **leading edge**입니다.

@(vif_ls_io.ev_spiclk_q);

toggle 후 event를 기다립니다.

if(!cpha | !sdr)

현재 코드에서는 CPHA=0일 때 leading edge 이후 data를 바꿉니다.

if(cb < data.size()) begin

queue boundary를 확인합니다.

set_drv_data(data[cb], cbit);

다음 data bit를 drive합니다.

cbit -= channel_width;

다음 bit 위치로 이동합니다.

end

첫 번째 data update block 종료입니다.

if (cbit < 0) begin

현재 byte의 모든 bit를 다 보냈는지 확인합니다.

cbit = 7;

다음 byte를 위해 bit index를 MSB로 초기화합니다.

cb += 1;

다음 byte로 이동합니다.

end

byte advance block 종료입니다.

@(vif_ls_io.ev_spiclk_q);

두 번째 clock edge 전 event를 기다립니다.

vif_ls_io.pad_lspi_sck = !vif_ls_io.pad_lspi_sck;

SCK를 다시 toggle합니다.  
CPOL=0이면 이 edge는 negedge, CPOL=1이면 posedge입니다. 즉 **trailing edge**입니다.

@(vif_ls_io.ev_spiclk_q);

toggle 후 event를 기다립니다.

if(cpha | !sdr) begin

현재 코드에서는 CPHA=1일 때 trailing edge 이후 data를 바꿉니다.

set_drv_data(data[cb], cbit);

다음 data bit를 drive합니다.  
여기는 `cb < data.size()` guard가 없어서 마지막에 out-of-range 가능성이 있습니다.

cbit -= channel_width;

다음 bit 위치로 이동합니다.

end

두 번째 data update block 종료입니다.

end

`repeat(ncks)` 종료입니다.

if (close_chn) begin

transaction을 닫을지 확인합니다.

set_drv_data(0, -1);

drive data를 clear하거나 Hi-Z 처리하는 용도로 보입니다.  
`set_drv_data()` 내부 구현에 따라 의미가 결정됩니다.

vif_ls_io.pad_lspi_csn = 1;

CSN을 high로 올려 SPI transaction을 종료합니다.

repeat(min_csn_high) @(vif_ls_io.ev_spiclk_q);

CSN high minimum timing만큼 기다립니다.

vif_ls_io.en_spiclk = 0;

SPI clock enable을 끕니다.

end

close block 종료입니다.

endtask

task 종료입니다.

핵심 문제는 이겁니다.

현재 CPOL=0 기준:

CPHA=0: posedge 후 data 변경  
CPHA=1: negedge 후 data 변경

일반 SPI 기준으로 data change는 보통 반대입니다.

CPHA=0: negedge 후 다음 data 변경  
CPHA=1: posedge 후 data 변경

그래서 이전에 말한 것처럼 두 조건을 바꾸는 게 맞습니다.