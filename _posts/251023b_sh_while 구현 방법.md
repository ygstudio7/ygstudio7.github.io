## 🧩 간단한 예제 구현

아래는 `run_tb`처럼 동작하는 축약 버전입니다.

```
#!/bin/sh

log="sim.log"
seed="random"
vargs=""

my_arg=true
while [ "$my_arg" = true ]; do
  case "$1" in
    -log)
      shift
      [ $# -eq 0 ] && echo "Missing log filename" && exit 1
      log="$1"
      shift
      ;;
    -seed)
      shift
      [ $# -eq 0 ] && echo "Missing seed number" && exit 1
      seed="$1"
      shift
      ;;
    -fsdb)
      vargs="$vargs +vcs+fsdbon"
      shift
      ;;
    *)
      my_arg=false
      ;;
  esac
done

echo "LOG FILE: $log"
echo "SEED: $seed"
echo "Remaining args: $*"
```

---

## ✅ 핵심 포인트 정리

| 요소                        | 설명                                |
| ------------------------- | --------------------------------- |
| `shift`                   | `$1`을 버리고 다음 인자로 이동               |
| `$#`                      | 남은 인자 개수                          |
| `$1`                      | 현재 첫 번째 인자                        |
| `case ... esac`           | 옵션별 분기 처리                         |
| `while [ "$#" -gt 0 ]`    | 남은 인자가 있을 때 반복                    |
| `break` 또는 `my_arg=false` | 루프 종료                             |
| *)                        | switch 문에서 기타에 해당 (anything else) |



## case 문

case문은 아래와 같이 사용한다.

```
case 변수 in
  패턴1)
    명령1
    ;;
  패턴2)
    명령2
    ;;
  *)
    기본_명령
    ;;
esac
```

즉,

- `패턴1`, `패턴2` 등은 `$1`의 값(즉, 인자)에 따라 매칭

- `*` 는 “그 외 모든 값(anything else)”을 의미하는 **wildcard**  
  (`*`는 `?`, `[]`처럼 glob 패턴 매칭에 쓰임)

- `;;` 은 각 case 블록의 끝을 의미함



## 🧠 run_tb 스크립트에서의 실제 역할

run_tb의 while 부분을 보면 이런 식이었죠:

`while [ "${my_arg}" = "true" ] ; do   case "$1" in     -log) shift; log=$1; shift ;;     -seed) shift; seed=$1; shift ;;     -fsdb) shift; vargs="${vargs} +vcs+fsdbon ..."; ;;     *)       my_arg=false       ;;  esac done`

여기서 `*)` 블록의 의미는:

> 위에서 정의된 모든 옵션(`-log`, `-seed`, `-fsdb` 등)에 **해당하지 않는 경우**,  
> 즉, **알 수 없는 인자**(예: `tb.v`, `+runtest=test_comm_err`)가 나오면  
> 루프를 멈추겠다.

즉, `*)`는 **“default / else 블록”** 역할이에요.
