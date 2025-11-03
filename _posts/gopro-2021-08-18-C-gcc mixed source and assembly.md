# Generating mixed source and assembly list using GCC

: GCC에서 소스 코드와 어셈블리 생성

GCC에서 소스 코드와 어셈블리를 혼합하여 disassemble 정보를 생성하는 옵션은 아래와 같다.



gcc사용한 컴파일시 최적화 옵션 `-O`와 `-g`를 사용하여, gdb 를 사용하여 디버깅에 필요한 정보를 생성한다.

 `-g`옵션은 소스 코드 컴파일시 gdb가 사용할 수 있는 디버깅 정보를 생성한다.



objdump를 사용하여 dissemble할 때, `-DSsx` 혹은 ``dSsx`와 같은 옵션을 사용하면, 디버깅에 필요한 모든 정보를 출력할 수 있다. 

`-d`: instruction에 대해 assembler mnemonics를 출력한다.

`-D`: instruction을 포함하는 것 뿐만 아니라 모든 section에서 disassembly 를 생성한다.

`-S`: 소스 코드를 disassembly 와 혼합해서 출력한다. 

`-s`: 모든 section의 전체 내용을 출력한다.

`-x`: symbol 테이블과 relocation entries를 포함한 모든 header 정보를 출력한다. 



makefile 구현시, 옵션들은 다음과 같이 추가하면 된다. 

```makefile
CFLAGS += -Os -g
DFLAGS := -DSsx

%.o: %.c $(hdrs)
	$(CC) $(CFLAGS) -o $@ -c $<
	
%.dump: %.elf
	$(OBJDUMP) $(DFLAGS) $< > $@
```





# Reference

objdump: https://ftp.gnu.org/old-gnu/Manuals/binutils-2.12/html_node/binutils_6.html