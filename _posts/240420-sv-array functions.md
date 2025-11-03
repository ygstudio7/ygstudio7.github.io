# Array system functions



SystemVerilog는 배열의 크기를 직접 선언하지 않은 채, 배열을 쉽게 조작할 수 있는 여러 특수한 시스템 함수들을 제공합니다. 합성 가능한 배열 쿼리 함수들에는 $left(), $right(), $low(), $high(), $increment(), $size(), $dimensions(), 및 $unpacked_dimensions() 이 있습니다. 다음은 이러한 함수 중 일부를 사용하는 예입니다:

```
typedef logic [31:0] d_array_t [0:4][0:4];
function d_array_t filter (
	input d_array_t d
);
	for (int i = $low(d,1); i <= $high(d,1); i++) begin: for_loop_on_i
		for (int j = $low(d,2); j <= $high(d,2); j++) begin: for_loop_on_j
			// ... perform some sort of operation on each element of d
		end: for_loop_on_j
	end: for_loop_on_i
	
	return d; // function return is an array
endfunction

```

