err 타겟 만들기



grep은 입력으로 전달된 파일의 내용에서 특정 문자열을 찾고자할 때 사용하는 리닉스 명령이이다.

리늑스에서 가장 많이 사용하는 명령어중의 하나이다.



grep 명령어가 문자열을 찾는 기능을 수행할때, 단순히 문자열이 일치하는 지 여부만을 확인하는 것이 아니라, 패턴 매칭을 할 수 있는 복잡한 정규 표현식 (Regular expression)을 지원한다.





### 예제 2: EN677MPW에 있는 warning과 err타겟 만들기

(2, 7) warning, err는 실제로 만들어질 목적 파일이 아니므로, (1,7)과 같이 target을 phony target으로 만들어야 한다.

(9)  `$(LOG_DIR)/xmvlog*` 파일에서 `\E`를 찾아서 `$(LOG_DIR)/error.log`에 저장한다.

```makefile
.PHONY: warning
warning:
	@grep -H -E "\*W," $(LOG_DIR)/xmvlog* | tee -i $(LOG_DIR)/warning.log
	@grep -H -E "\*W," $(LOG_DIR)/xmelab* | tee -i -a $(LOG_DIR)/warning.log
	@grep -H -E "\*W," $(LOG_DIR)/xmsim* | tee -i -a $(LOG_DIR)/warning.log

.PHONY: err
err:
	@grep -H -E "\*E," $(LOG_DIR)/xmvlog* | tee -i $(LOG_DIR)/error.log
	@grep -H -E "\*E," $(LOG_DIR)/xmelab* | tee -i -a $(LOG_DIR)/error.log
	@grep -H -E "\*E," $(LOG_DIR)/xmsim* | tee -i -a $(LOG_DIR)/error.log
```



### 예제 1: 텍스트 파일에서 `\Error` 문자열 검색

```bash
grep -H -E "\%Error" bld_Unittest_encor_unit_DvTbIspTestConfig_run-binary-debug_0a.txt
```

결과:

```bash
bld_Unittest_encor_unit_DvTbIspTestConfig_run-binary-debug_0a.txt:%Error-TIMESCALEMOD: /hdd2/ygkim/riscv/chipyard/encor-chipyard15_210914/encor-chipyard15/sims/verilator/generated-src/encor.test.unittest.DvTbIspTestConfig/tb_isp.v:361:9: Timescale missing on this module as other modules have it (IEEE 1800-2017 3.14.2.2)
bld_Unittest_encor_unit_DvTbIspTestConfig_run-binary-debug_0a.txt:%Error-TIMESCALEMOD: /hdd2/ygkim/riscv/chipyard/encor-chipyard15_210914/encor-chipyard15/sims/verilator/generated-src/encor.test.unittest.DvTbIspTestConfig/tb_isp.v:1245:12: Timescale missing on this module as other modules have it (IEEE 1800-2017 3.14.2.2)
bld_Unittest_encor_unit_DvTbIspTestConfig_run-binary-debug_0a.txt:%Error: Exiting due to 2 error(s)

```

![image-20211014094314502](E:\_ygkim\_Doc\_future\_work\github pages\ygstudio7\ygstudio7.github.io\_posts\2021-10-14-makefile-err 타겟 만들기.assets\image-20211014094314502.png)



### [ ] grep 명령어 옵션

grep 명령어 옵션은 `grep --help`명령을 통해 확인할 수 있다.

예제 1에서 사용한 `-H`는 검색결과 출력 라인 앞에 입력된 파일 이름을 표시한다.

또한 위에서 사용한 `-E`는 확장된 정규 표현식을 사용한다.

```
    grep [OPTION...] PATTERN [FILE...]
        -E        : PATTERN을 확장 정규 표현식(Extended RegEx)으로 해석.
        -F        : PATTERN을 정규 표현식(RegEx)이 아닌 일반 문자열로 해석.
        -G        : PATTERN을 기본 정규 표현식(Basic RegEx)으로 해석.
        -P        : PATTERN을 Perl 정규 표현식(Perl RegEx)으로 해석.
        -e        : 매칭을 위한 PATTERN 전달.
        -f        : 파일에 기록된 내용을 PATTERN으로 사용.
        -i        : 대/소문자 무시.
        -v        : 매칭되는 PATTERN이 존재하지 않는 라인 선택.
        -w        : 단어(word) 단위로 매칭.
        -x        : 라인(line) 단위로 매칭.
        -z        : 라인을 newline(\n)이 아닌 NULL(\0)로 구분.
        -m        : 최대 검색 결과 갯수 제한.
        -b        : 패턴이 매치된 각 라인(-o 사용 시 문자열)의 바이트 옵셋 출력.
        -n        : 검색 결과 출력 라인 앞에 라인 번호 출력.
        -H        : 검색 결과 출력 라인 앞에 파일 이름 표시.
        -h        : 검색 결과 출력 시, 파일 이름 무시.
        -o        : 매치되는 문자열만 표시.
        -q        : 검색 결과 출력하지 않음.
        -a        : 바이너리 파일을 텍스트 파일처럼 처리.
        -I        : 바이너리 파일은 검사하지 않음.
        -d        : 디렉토리 처리 방식 지정. (read, recurse, skip)
        -D        : 장치 파일 처리 방식 지정. (read, skip)
        -r        : 하위 디렉토리 탐색.
        -R        : 심볼릭 링크를 따라가며 모든 하위 디렉토리 탐색.
        -L        : PATTERN이 존재하지 않는 파일 이름만 표시.
        -l        : 패턴이 존재하는 파일 이름만 표시.
        -c        : 파일 당 패턴이 일치하는 라인의 갯수 출력.
```



### [ ] 정규 표현식

정규 표현식 (Regular Expression)은, 특정 규칙을 가진 문자열 집합을 표현하기 위해 사용되는 형식 언어로써, 주로 문자열의 패턴 매칭이나 문자열을 치환하기 위해 사용된다.

문자열 검색시 정규 표현식을 사용하게 되면, 지정된 문자열의 문자가 같은지 여부뿐만 아니라, 정규 표현식에 의한 규칙이 매칭되는지 여부를 확인할 수 있다.



| 메타 문자 (Meta Character) | 설명                                         |
| -------------------------- | -------------------------------------------- |
| .                          | 1개의 문자 매치 (정확히 1개의 문자와 매치)   |
| *                          | 앞 문자가 0회 이상 매치                      |
| {n}                        | 앞 문자가 정확히 n회 매치                    |
| {n,m}                      | 앞 문자가 n회 이상 m회 이하 매치             |
| [ ]                        | 대괄호에 포함된 문자 중 한개와 매치          |
| [^ ]                       | 대괄호 안에서 ^뒤에 있는 문자들을 제외       |
| [ - ]                      | 대괄호 안 문자 범위에 있는 문자들 매치       |
| ()                         | 표현식을 그룹화                              |
| ^                          | 문자열 라인의 처음                           |
| $                          | 문자열 라인의 마지막                         |
| ?                          | 앞 문자가 0 또는 1회 매치 (확장 정규 표현식) |
| +                          | 앞 문자가 1회 이상 매치 (확장 정규 표현식)   |
| \|                         | 표현식 논리 OR (확장 정규 표현식)            |



## TAG

리눅스 grep, 정규 표현식, makefile 타겟, 리눅스 문자열 검색, 리눅스 패턴 매칭, 

# 참고

https://recipes4dev.tistory.com/157