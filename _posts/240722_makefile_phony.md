## **.PHONY를 쓰는 첫번째 이유: 실제 파일이름과의 충돌을 해결**

 

Phony는 가짜라는 의미이며 phony target 이란 실제 파일이름이 아닌 target을 의미한다.

make 명령이 실행되는 디렉토리에 Makefile의 target과 같은 이름의 파일이 존재할 경우에 충돌이 발생하는데 .PHONY에 명시하여 이를 회피할 수 있다.

 

**만약 test라는 파일이 있는 디렉토리에서 make test 라고 파일이름과 같은 target인 test를 실행하면 어떻게 될까?**

 

.PHONY에 명시한 경우 아래와 같이 정상적으로 실행된다.

```
# .PHONY를 사용하는 Makefile
.PHONY: test
test:
  @echo test makefile
```

Shell에서 실행 결과

```
→ make test
test makefile
```

 

**.PHONY에 명시하지 않은 경우 test라는 파일을 인식하고 Makefile의 target 내용이 실행되지 않는다.**

```
# .PHONY를 사용하지 않는 Makefile
test:
  @echo test makefile
```

Shell에서 실행결과 

```
→ make test
make: `test'는 이미 갱신되었습니다.
```

##  

## **.PHONY를 쓰는 두 번째 이유: 성능 개선**

 

Makefile 안에서 make를 다시 실행시키는 경우를 생각해보자.

아래 링크에서 잘 설명되어 있으며 여기의 예제를 가져다 설명해본다.

\- 간결한 예제 링크: https://sodocumentation.net/makefile/topic/5542/-phony-target

 

```
/main
     |_ Makefile
     |_ /foo
            |_ Makefile
            |_ ... // other files
     |_ /bar
            |_ Makefile
            |_ ... // other files
     |_ /koo
            |_ Makefile
            |_ ... // other files
```

 

**main 디렉토리의 Makefile이 아래와 같을 때에 $make subdirs를 실행시키면 어떻게 될까?**

```
SUBDIRS = foo bar koo

subdirs:
        for dir in $(SUBDIRS); do \
          $(MAKE) -C $$dir; \
        done
```

 

두 가지 문제점이 있다.

1. 하위 make에서 에러가 발생해도 계속 빌드가 진행된다.
2. 하나의 규칙만이 실행되기에 make의 장점인 병렬 수행이 되지 않는다.

 

 

**이렇게 .PHONY를 이용하면 위의 문제들이 모두 해결된다.**

 

```
SUBDIRS = foo bar koo

.PHONY: subdirs $(SUBDIRS)
subdirs: $(SUBDIRS)

$(SUBDIRS):
        $(MAKE) -C $@
```
