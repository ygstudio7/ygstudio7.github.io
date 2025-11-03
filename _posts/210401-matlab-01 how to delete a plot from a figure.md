# Plot



## How to delete a figure from a plot



plot후 반환된 handle을 저장한후, 이 handle을 인자로 delete함수를 호출하면 된다.

The `plot` function returns a handle to the plotted object. 

Call `delete` with this.

i.e.,

```matlab
figure
h = plot([x1 x2], [y1 y2])
...
delete(h)
```







## Reference

https://www.mathworks.com/matlabcentral/answers/315848-deleting-individual-plots-from-a-figure



