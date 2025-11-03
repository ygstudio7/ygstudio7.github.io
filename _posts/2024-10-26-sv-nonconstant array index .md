참고: https://verificationacademy.com/forums/t/error-a-variable-index-into-the-generate-block-reg-array-is-illegal/35016



# My example

	test_intf A(), B[N]();


아래 코드는 SEL이 nonconstant라서 오류 발생

```
assign A = B[SEL];
```



always_comb와 case statement를 사용해서 구성할 수 있지만, parameterize하게 구성할 수 없는 단점이 있다.

```
	always_comb begin
	case(SEL)
	0: A = B[0];
	1: A = B[1];
	2: A = B[2];
	default: A = B[3];
	endcase
	end
```



아래 코드는 동작하지 않는다.

```
always_comb begin
	for(integer i=0; i<N; i++)
		if(i==SEL)
			A = B[SEL];
end	
```



아래와 같이 generate-loop아래 always 문을 넣어야 동작한다.


	for(genvar i=0; i<N; i++)
	always_comb begin
		if(i==SEL) 
			A = B[SEL];
	end

