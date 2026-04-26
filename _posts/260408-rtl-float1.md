
# 기본
## 1. ASCII 형태로 저장된 경우
* ASCII 형태: 1.234e-02, 0.3, -0.23
* Modelsim은 기본적으로 textio 같은 패키지 사용
	* textio.read(real'(...)) 사용 
* $fopen, $fscanf, $fread같은 파일 I/O 사용
	* $fscanf("%f")로 읽어서 실수형 (real/shortreal) 변수에 저장

## 2. Binary float 저장파일의 경우 
* IEEE754 32b/64b binary 로 저장된 경우, endian과 정밀도 (32b/64b)을 맞춘다
* $fread로 raw로 읽기 --> shortreal/real로 캐스팅
* 연구/검증단계에서는 csv나 ASCII가 훨씬 편하다



# Project 1: ASCII file

```
modelsim_proj/
├─ rtl/
│  └─ dut.sv
├─ tb/
│  ├─ tb_top.sv
│  └─ floats.txt
└─ scripts/
   ├─ new_project.do
   ├─ compile.do
   ├─ sim.do
   └─ wave.do
```

### rtl/dut.sv: 간단한 DUT로 입력 실수값을 증폭

```verilog
module dut;
  // 이 모듈은 포트 없이, 태스크/함수로만 TB에서 호출하는 형태의 예시입니다.
  // 실제로는 고정소수점/정수 인터페이스를 두고 변환하여 사용하세요.

  // 실수 값을 받아 scale 배를 곱해 반환
  function real scale_real(input real x, input real scale = 2.0);
    return x * scale;
  endfunction

endmodule
```

### tb_top.sv
ASCII float 파일 읽어서 DUT함수에 적용
```verilog
module tb_top;
  import "DPI-C" function void sv_fatal(string msg="");

  integer fd;
  string  line;
  real    val, outv;
  real    sum = 0.0;
  int     cnt = 0;

  // DUT 인스턴스
  dut u_dut();

  initial begin
    // 파일 열기
    fd = $fopen("tb/floats.txt", "r");
    if (fd == 0) begin
      $error("cannot open tb/floats.txt");
      $finish;
    end

    // 한 줄씩 읽어서 실수 파싱
    while ($fgets(line, fd)) begin
      // 주석/빈줄 스킵
      if (!line.len()) continue;
      if (line.tolower().substr(0,0) == "#") continue;

      if ($sscanf(line, " %f ", val) == 1) begin
        outv = u_dut.scale_real(val, 1.5); // 예: 1.5배 스케일
        sum += outv;
        cnt++;
        // $display("in=%f out=%f", val, outv);
      end
    end
    $fclose(fd);

    $display("read %0d floats, sum(out)=%f, avg(out)=%f",
             cnt, sum, (cnt ? sum/cnt : 0.0));

    // 파형 관찰을 위해 약간 대기 후 종료
    #10 $finish;
  end
endmodule

```

### 입력 파일: floats.txt
```
# sample floats
0.5
1.0
-2.75
3.14e-1
10
```


# Project 2: IEEE754 binary

### MATLAB 에서 파일 생성
: float32, LE 방식
```matlab
x = single([0.5 1.0 -2.75 0.314 10]);  % 예시 데이터
fid = fopen('data_f32_le.bin','w','ieee-le');   % little-endian
fwrite(fid, x, 'single');
fclose(fid);
```
: float64
```matlab
y = [0.5 1.0 -2.75 0.314 10];          % double 기본
fid = fopen('data_f64_le.bin','w','ieee-le');
fwrite(fid, y, 'double');
fclose(fid);
```

: length 정보 넣기: uint32
```matlab
vals = single(rand(100,1));
fid = fopen('data_with_header.bin','w','ieee-le');
fwrite(fid, uint32(numel(vals)), 'uint32');   % 헤더: N
fwrite(fid, vals, 'single');                  % 본문: IEEE754
fclose(fid);
```


### Msim에서 length 포함해서 읽기 (LE)
```verilog
module read_f32_with_header;
  integer fd;
  int unsigned N, w32;
  shortreal sx;
  shortreal data[$];
// (5)
  initial begin
    fd = $fopen("data_with_header.bin", "rb");
    if (fd == 0) $fatal("open failed");

    if ($fread(N, fd) != 4) $fatal("failed to read N"); // (10)
    for (int i = 0; i < N; i++) begin
      if ($fread(w32, fd) != 4) $fatal("EOF @i=%0d", i);
      sx = $bitstoshortreal(w32);
      data.push_back(sx);
    end // (15)
    $fclose(fd);
    $display("N=%0d read=%0d", N, data.size());
    #1 $finish;
  end
endmodule
```
(10) length 정보 N 읽기
(11) N개 동안 
(12) fread로 4bytes씩 읽어서 w32에 저장
(13) bitstoshortreal함수를 이용해서 w32를 32b float 인 sx로 변환
(14) sx를 data queue 에 넣기

만약 BE로 썼다면, 다음과 가티 swap하는 함수를 사용한다.
```verilog
function automatic int unsigned bswap32(input int unsigned x);
  return {x[7:0], x[15:8], x[23:16], x[31:24]};
endfunction

function automatic longint unsigned bswap64(input longint unsigned x);
  return {x[7:0], x[15:8], x[23:16], x[31:24],
          x[39:32], x[47:40], x[55:48], x[63:56]};
endfunction

// 사용 예:
// w32 = bswap32(w32);
// w64 = bswap64(w64);

```
# Project 3: 실제

### matlab
hdr 프로젝트에서 ir_imwriteb를 만들고, line 3와 같이 4x5x3만 임시로 저장할 수 있도록 파일을 구현했고, 
```matlab
function psnr = ir_imwriteb(I, name)

% I = I(1:4,1:5,:)*1000; % FIXME: test only
[VW HW ZW] = size(I);
// (5)
fid = fopen([name, '.bin'],'w','ieee-be');   % big-endian == SV

J = permute(I,[3 2 1]);
% disp(J(:)); % FIXME: test only
// (10)
fwrite(fid, uint32(VW), 'uint32');
fwrite(fid, uint32(HW), 'uint32');
fwrite(fid, uint32(ZW), 'uint32');
fwrite(fid, J, 'single'); % IEEE754

fclose(fid);
```

(6) 파일을 'w' binary이면서 BE로 열고, 
(8) permute해서 순서를 바꾼다. (3) 에서 4x5x3 행렬이면, 3x5x4가 된다. 그 이유는 픽셀 단위로 저장해서, rtl에서 쉽게 픽셀 단위로 가져오려고 한다. 두번째는 HW이고, 마지막이 VW가 된다. 따라서, 픽셀 단위로 raster scan 순서가 된다
(11-13)  VW, HW, ZW를  uint32로 저장하고
(14) IEEE754 single precision으로 저장한다


### rtl

```verilog
module tb_top;
  integer fd;
  int unsigned VW, HW, ZW, w32;
  shortreal sx;
  shortreal data[$];
// (5)
  initial begin
    fd = $fopen("bistr_0120_0068.bin", "rb");
    if (fd == 0) $fatal("open failed");

    if ($fread(VW, fd) != 4) $fatal("failed to read VW"); // (10)
    if ($fread(HW, fd) != 4) $fatal("failed to read HW");
    if ($fread(ZW, fd) != 4) $fatal("failed to read ZW");
    $display("[VW HW ZW] = [%0d %0d %0d]", VW, HW, ZW);

    for (int i=0; i<VW; i++) begin // (15)
    	for (int j=0; j<HW; j++) begin
    		for (int k=0; k<ZW; k++) begin
      		if ($fread(w32, fd) != 4) $fatal("EOF @i=%0d", i);
		      sx = $bitstoshortreal(w32);
		      data.push_back(sx); // (20)
		    end
    	end
    end
    $fclose(fd);
    // (25)
    $display("read=%0d", data.size());
    foreach(data[i])
    	$display("data[%d] = %f", i, data[i]);
    
// (30)
    //#1 $finish;
  end
endmodule
```
(7) 위에서 저장한 4x5x3 파일을 rb 로 열고
(10-12) VW, HW, ZW를 읽어들이고
(15-17) ZW, HW, VW  의 raster scan 순으로 읽어서
(19) bitstoshortreal을 사용해 float으로 변경후, data queue 에 저장
(26) data.size 출력
(27) 각 data[i] 를 출력한다.

