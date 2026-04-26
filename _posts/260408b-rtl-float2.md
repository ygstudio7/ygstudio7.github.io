
## 1. 전체 검증 구조
* MATLAB
	* 2개 버전으록 구성
		* 알고리즘용
		* RTL 검증용: RTL-equivalent
	* 입력 영상 저장
		* input.[txt|hex|flt]
		* 저장 포맷은 32b hex text로 한다. 32b IEEE754로 그대로 변환 가능
	* 각 알고리즘 stage-by-stage 중간 결과 저장
		* stg1_golden.[txt|hex|flt]
		* stg2_golden.[txt|hex|flt]
	* 최종 출력 저장
		* final.[txt|hex|flt]
- RTL
	- RTL data는 32bit IEE754로
	- 위 입력 영상을 읽어 DUT에 주입
	- 각 알고리즘 stage중간 결과를 읽음
	- 각 stage 마다 MATLAB golden 과 비교
- TB
	- sb에서 1차 비교한다.
		- bit-exact?
		- tolerance 기반
	- file read
	- pixel driver
	- output monitor
	- scoreboard 
- MATLAB
	- RTL 결과를 matlab으로 가져와서 2차 시각화


## 저장 포맷
IEEE single은 float32임
### 방식 1: binary raw float32
각 픽셀을 4byte IEEE754 single로 저장
(+) matlab 저장 용이, 크기 작음, 실제 single 그대로 유지
(-) RTL 읽을때 약간 귀찮고, endian 확인 필요

### 방식 2: hex ascii text 파일
각 픽셀을 4byte hex ascii로 저장
(+) RTL 에서 $readmemh로 읽기 편함, 
(-) 파일 크기 커짐, matlab에서 변환 필요

### matlab code
[W] 여기에 H, W 정보도 같이 넣으면 좋다.

matlab
```matlab
function save_single_image_hex(fname, img)
	% row-major
	if ~isa(img, 'single')
	  img = single(img);
	end
	
	img_linear = img.'; % row-major WxH
	img_linear = img_linear(:); 
	
	u32 = typecast(img_linear, 'uint32');
	
	fid = fopen([fname, '.hex'], 'w');
	assert(fid ~= -1, 'cannot open file: %s', fname);
	
	fprintf(fid, '%08x\n', u32);
	fclose(fid);
end

```

```matlab
function save_single_image_bin(fname, img)
	% row-major
	if ~isa(img, 'single')
	  img = single(img);
	end
	
	img_linear = img.'; % row-major WxH
	img_linear = img_linear(:); 
		
	fid = fopen([fname, '.bin'], 'wb');
	assert(fid ~= -1, 'cannot open file: %s', fname);
	
	fwrite(fid, img_linear, 'single');
	fclose(fid);
end

```


```matlab
function save_single_image(fname, img, fmt)

if(fmt=='hex')
	save_single_image_hex(fname, img);
else if(fmt=='bin')
	save_single_image_bin(fname, image)
end

end

```

```matlab
fmt = 'bin'; % 'hex'
save_single_image('input', input_img, fmt);
save_single_image('s1_golden', s1_img, fmt);
save_single_image('s2_golden', s1_img, fmt);
save_single_image('final', final_img, fmt);
```



rtl
```



logic [31:0] in_mem[0:N-1];
logic [31:0] s1_golden_mem[0:N-1];

// load
initial begin
	$readmemh("input.hex:, in_mem);
	$readmemh("s1_golden.hex:, s1_golden_mem);
end

// input
logic [31:0] in_val;
shortreal in_fval;

always_comb begin
	in_val = s1_golden_mem[p];
	in_fval = $bitstoshortreal(in_val);
end

```

rtl bit 단위 비교
```
if(s1_val != s1_golden_mem[p]) begin
	$display("Mismatch at %0d: dut=%08x golden=%08x", 
			p, s1_bits, s1_golden_mem[p]);
end
```

rtl tolerance 비교
```
shortreal err_tol = 1e-5;
shortreal s1_fval, s1_golden_fval;

s1_fval = $bitstoshortreal(s1_val);
s1_golden_fval = $bitstoshortreal(s1_golden_val);
err = (s1_fval > s1_golden_fval) ? (s1_fva ㅁㅋl - s1_golden_fval) : (s1_golden_fval - s1_fval);

if(err > err_tol) begin
  $display("Mismatch at %0d: dut=%f golden=%f",
		  p, s1_fval, s1_golden_fval);
end

```


## 검증 전략


1. 입력 파일 검증: endian, line order 등
2. stage 별 검증

	예: 각 블럭마다
	* 독립 입력
	* 독립 golden
	* block-level TB
	이렇게 있으면 전체 시스템 디버깅이 쉬워진다

3. 전체 full pipeline 검증
	전체 입력 영상을 넣고,
	각 stage output 을 비교
	최종 output 비교

[W] 지금 생각하지 못했던 부분은 각 블럭 마다 독립 입력, 독립 golden과 tb가 있으면 s블럭별 구현이 가능하므로 개발이 좀더 쉬워질 것 같다

###  테스트
페턴, 렌덤 테스트와 실제 영상 테스트를 둘다 수행

* 패턴 테스트: Corner case용
	* All zero
	* All one
	* Ramp
	* Checkerboard
	* Impulse
	* 랜덤 패턴
* 실제 영상
* 해상도 변경 테스트
	* 8x8
	* 16x16
	* 64x64
	* 최종 크기 영상
* 

### RTL Interface
부동 소숫점 single이면, RTL 포트도 가능하면 이렇게 하는게 좋다
float을 real/shortreal로 직접 주지 말고, 32 bit logic으로 전달하는게 편리하다
검증 관점에서 단순하다

```verilog
input logic in_valid,
output logic in_ready,
input logic [31:0] in_data, // IEEE754 single

output logic out_valid,
input logic out_ready,
output logic [31:0] out_data
```
## 중간 결과 비교를 위한 RTL 내부 TAP 구현

중간 결과를 확인하려면 DUT 내부 stage 결과를 tb가 볼 수 있어야 한다
보통 2가지 방법을 사용한다. (IJVM 아니면)

### 방법 1: debug port 추가
(+) 명확하고 비교 쉬움
(-) 포트가 늘어남 ->  interface로 하면 ok
예:
- debug tap 

### 방법 2: hierarhical reference
TB에서 내부 신호를 직접 참조
(+) DUT I/F 변경 불필요
(-) 이식성X : 구조 바뀌면 계속 변경해야함 


### Latency
RTL pipeline latency때문에 golden index와 DUT에서의 index를 맞춰야 한다
[Q] 채집사에게 좋은 방법이 있는지 물어본다. 


## 간단 TB 구조 1

```verilog
module tb;

parameter int H = 4;
parameter int W = 5;
parameter int NP = H * W;

logic clk, rstn;

logic in_valid, in_ready;
logic [31:0] in_data;

logic out_valid, out_ready;
logic [31:0] out_data;

logic [31:0] input_mem [0:NP-1];
logic [31:0] stg1_golden_mem[0:NP-1];
logic [31:0] final_golden_mem [0:NP-1];

int in_ix, out_idx, err_cnt;

// DUT
dut u_dut (
	.clk      (clk),
	.rstn     (rstn),
	
	.in_valid (in_valid),
	.in_ready (in_ready),
	.in_data  (in_data),
	
	.out_valid (out_valid),
	.out_ready (out_ready),
	.out_data  (out_data)
)l

// file load
initial begin
  $readmemh("input.hex", input_mem);
  $readmemh("stg1.hex", stg1_golden_mem);
  $readmemh("final.hex", final_golden_mem);
end

// clock
initial begin
  clk = 0;
  forever #5 clk = ~clk;
end

// reset
initial begin
  rstn = 0;
  repeat (10) @(posedge clk);
  rstn = 1;
end

// input driver
always @(posedge clk) begin
	if(!rstn) begin
		in_valid <= 0;
		in_data <= '0;
		in_idx = 0;
	end
	else if(in_idx < NP) begin
		in_valid <= 1;
		in_data M= input_mem[in_idx];
		if(in_valid && in_ready)
			in_idx <= in_idx + 1;
	end else
		in_valid <= 0;
	end
end

// output monitor / scorboard
initial begin
  out_ready = 1;
end

always @(posedge clk) begin
  if(rstn && out_valid && out_ready) begin
    if(out_data !== golden_mem[out_idx]) begin
      $display("Mismatch at %0d": dut=%08x golden=%08x?, out_idx, out_data, golden_mem[out_idx]);
      err_cnt <= err_cnt + 1;
    end
    out_idx <= out_idx+ 1;
    
    if(out_idx == NP-1) begin
	    $display("Simulation done. err_cnt=%0d"m err_cnt);
	    #20;
	    $finish;
	end
  end
end

// s
endmodule

```


### Pass-through DUT 1차 검증
처음부터 알고리즘을 넣지 말고, DUT를 아래처럼 그대로 출력하도록 만들어서 파일 I/O와 순서가 맞는지 먼저 확인한다.
[CG] 아래 코드는 다시 만들어달라고 해야 겠네

```verilog
module dut (
	input logic clk,
	input logic rstn,
	
	input logic in_valid, 
	output logic in_ready,
	input logic [31:0] in_data,
	
	output logic out_valid,
	input logic out_ready,
	output logic [31:0] out_data
);

assign in_ready = out_ready || !out_valid;

always_ff @(posedge clk or negedge rstn) begin
	if(!rstn) begin
		out_valid <= 0;
		out_data <= '0;
	end else if(in_ready) begin
		out_valid <= in_valid;
		out_data <= in_data;
	end
end

endmodule
```

### 중간 stage 비교용 debug TAP

모든 모듈에 같은 if를 쓸 수도 있지. dbg if.sp dbg;

```verilog
interface dbg_if (
	logic stg1_valid;
	logic [31:0] stg1_data;
	
	logic std2_valid;
	logic [31:0] stg2_data;
	
	modport stg1_mp (output stg1_valid, stg1_data);
	modport stg1_sp (input stg1_valid, stg1_data);
	
)
```


```verilog
module stage1 (
...
dbg_if.stg1_mp dbg
);

dbg.stg1_valid = ...;
dbg.stg1_data = ...;

endmodule



```

debug TAP을 출력해서 


## MATLAB 비교
RTL 결과를 파일로 dump하고, MATLAB에서 그 파일을 다시 읽어서 영상으로 재구성해 비교하면 mismatch 위치를 영상으로 볼 수 있어서 디버깅이 빨라질 수 있다.

예:
* RTL 출력: rtl_out.hex

```matlab
fid = fopen('rtl_out.hex','r');
u = scanf(fid, '%x');
fclose(fid);

u = uint32(u);
rtl_out = typecast(u, 'single');
rtl_out = reshape(rtl_out, H, W);

diff = abs(rtl_out - golden_out);
max(diff(:));
imagesc(diff); colorbar;

```
