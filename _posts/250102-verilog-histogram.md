# Histogram module

아래는 systemverilog로 히스토그램 계산 모듈을 작성한 예제입니다. 이 모듈은 입력 데이터를 수신하고, 각 값이 등장한 횟수를 계산하여 히스토그램을 생성합니다.

```verilog
//******************************************************************************
//	Copyright(c) Young Geun Kim 2003-
//
//	File Name:		histogram.sv
//	Description:	
//	Author:			Young Geun Kim (ygkim@eyenix.com;ygkim912@gmail.com)
//
//	Note:	
//
//	Ver Date		Author		Changes
//	--- ----		------		-------
//	0.1	250102		ygkim		first designed
// -----------------------------------------------------------------------------
module histogram #(
	parameter DATA_WIDTH = 8,
	parameter BIN_COUNT = 16
)(
	input logic 				clk,
	input logic 				rst_n,
	input logic [DATA_WIDTH-1:0] data_in,
	input logic 				valid_in,
	output logic [31:0] 		hbins[BIN_COUNT],
	output logic 				done
);

	// Internal variables
	logic [31:0]	hbin_temp[BIN_COUNT];
	integer 			i;
	
	always_ff @(posedge clk or negedge rst_n) begin
		if(!rst_n) begin
			for(i=0; i<BIN_COUNT; i++) begin
				hbin_temp[i] <= 32'd0;
				hbins[i] <= 32'd0;
			end
			done <= 1'b0;
		end
		else begin
			// Update hbins when valid data is received
			if(valid_in) begin
				if(data_in < BIN_COUNT) begin
					hbin_temp[data_in] ++;
				end
			end
		end
	end
	
	// Output
	always_ff @(posedge clk or negedge rst_n) begin
		if(!rst_n) begin
			done <= 1'b0;
		end
		else begin
			for(i=0; i<BIN_COUNT; i++) begin
				hbins[i] <= hbin_temp[i];
			end
			done <= 1'b1;
		end
	end

endmodule

```

이 코드는 다음과 같은 기능을 제공합니다.

1. 데이터 입력 및 유효성 검사: data_in 과 valid_in을 통해 입력 데이터를 수신하며, valid_in이 활성화된 경우 데이터를 처리합니다.
2. 히스토그램 계산: data_in의 값에 해당하는 bin을 증가시켜 히스토그램을 작성합니다.
3. 결과 출력: bins는 각 bin의 카운트를 나타내며, done신호는 계산 완료 상태를 나타냅니다.
4. 파라미터화: DATA_WIDTH와 BIN_COUNT를 사용하여 데이터 폭과 bin의 개수를 조정할 수 있습니다,

원하는 대로 추가적인 수정이나 가능을 추가할 수도 있습니다. 예를 들어, 특정 조건에 따라 히스토그램을 초기화하거나 결과를 더 정교하게 출력하도록 설정할 수 있습니다.



# Testbench

```verilog
`timescale 1ns/1ps

module tb_histogram;
    
	// Parameters
	parameter DATA_WIDTH = 8;
	parameter BIN_COUNT = 16;
	
	// Testbench signals
	logic 	clk;
	logic 	rst_n;
	logic 	[DATA_WIDTH-1:0] data_in;
	logic		valid_in;
	logic 	[31:0] hbins [BIN_COUNT];
	logic		done;
	
	// Instantiate the histogram module
	histogram #(DATA_WIDTH, BIN_COUNT) dut (
		.clk			(clk),
		.rst_n		(rst_n),
		.data_in		(data_in),
		.valid_in	(valid_in),
		.hbins		(hbins),
		.done			(done)
	);
	
	// Clock
	always #5 clk = ~clk;		// 10 ns clock period
	
	// Testbench
	initial begin
		// Initialize signals
		clk = 0;
		rst_n = 0;
		data_in = 0;
		valid_in = 0;
		
		// Reset
		#20 rst_n = 1;
		
		// Test case 1: feed data
		#10 send_data(8'd3);
		#10 send_data(8'd3);
		#10 send_data(8'd7);
		#10 send_data(8'd15);
		#10 send_data(8'd7);
		#10 send_data(8'd15);
		#10 send_data(8'd15);
		
		// Wait for calculation to complete
		#100;
		
		// Display histogram hbins
		// wait(done);
		display_bins();
		
		// Finish simulation
		#20 $finish;
	end

	
	// Task to send data
	task send_data(input [DATA_WIDTH-1:0] value);
		begin
			data_in = value;
			valid_in = 1;
			#10;
			valid_in = 0;
		end
	endtask
	
	// Task to display hbins
	task display_bins;
		integer i;
		begin
			$display("Histogram Bins:");
			for(i=0; i<BIN_COUNT; i++) begin
			$display("Bin[%0d] = %0d", i, hbins[i]);
		end
		end
	endtask
	
endmodule	
    
    
```

이 테스트벤치는 다음과 같은 작업을 수행합니다.

1. 클럭 생성: 10 ns 주기의 클럭 신호를 생성합니다.
2. 초기화 및 리셋: 모든 신호를 초기화하고 리셋 신호를 활성화한 뒤 비활성화합니다.
3. 데이터 전송: send_data 태스크를 사용해 히스토그램 모듈로 데이터를 전송합니다.
4. 결과 확인: display_bins 태스크를 사용해 각 bin값을 출력합니다.
5. 종료: 모든 작업이 완료되면 시뮬레이션을 종료합니다.

이 코드를 시뮬레이터에서 실행하면 히스토그램 모듈의 동작을 검증할 수 있습니다. 필요한 경우 추가 테스트 케이스를 추가하거나 출력 형식을 수정할 수 있습니다.



# Modelsim 시뮬레이션

Modelsim 에서 실행하기 위해 필요한 파일은 다음과 같습니다.

1. Top-level testbench: tb_histogram.v, 앞서 제공한 testbench 코드
2. Design Under Test (DUT): histogram.sv, 앞서 제공한 histogram 모듈 코드
3. Simulation Script (run_sim.tcl): 컴파일, 시뮬레이션, 결과 표시 등을 자동화하기 위한 스크립트

```tcl
# TCL script for ModelSim simulation

# Clear previous simulation data
vlib work
vmap work work

# Compile the design files
vlog histogram.sv
vlog tb_histogram.sv

# Run the simulation
vsim -c tb_histogram

# Show the waveforms
add wave -r /*
run -all

```



### 실행 방법

1. 위의 파일들을 각각 histogram.sv, tb_histogram.v, run_sim.tcl로 저장합니다.
2. Modelsim을 열고 시뮬레이션 디렉토리를 설정합니다.
3. TCL 스크립트를 실행합니다.

```bash
do run_sim.tcl
```

이 과정을 통해 설계 및 테스트를 컴파일하고, 시뮬레이션 결과를 확인할 수 있습니다. 필요하면 TCL 스크립트를 수정하여 시뮬레이션 조건을 조정합니다.









### 참고

chatgpt: histogram 구하는 모듈을 systemverilog로 만들어줘

chatgpt: 이 모듈을 테스트하는 testbench도 만들어줘

chatgpt: 이 testbench를 modelsim에서 실행하기 위한 파일들도 만들어줘