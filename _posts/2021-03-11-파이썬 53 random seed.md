참고: https://wikidocs.net/15652



seed함수를 사용하지 않으면 실행할때마다 동일한 값이 나온다.

random.choice()사용시





random.shuffle(list)는 list자체를 random하게 섞는다.



아래에서 a=None이면, 기본적으로 현재시간을 seed로 사용하에 random한 값을 만들 수 있다.

참고: https://pynative.com/python-random-seed/

```python
random.seed(a=None, version=2)
```



