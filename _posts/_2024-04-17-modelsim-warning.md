



# Question

The following messages are displayed when I run simulation with modelsim 6.0.

Warning: (vsim-PLI-3691): Expected a system task, not a system function '$fseek'.

Warning: (vsim-PLI-3691): Expected a system task, not a system function '$sscanf'.

Is this a bug of Modelsim 6.0, or did I miss some files or setting?



# Answer

$fseek as a task. It is a function, and returns a value. You must call it as a function.

In other words, you tried to do something like

```
begin
$fseek(fd,0,0);
end
```

```
begin
$sscanf(str,"%s_%d",name,no);
end
```



You must instead do something like

```
integer code;
begin
code = $fseek(fd,0,0);
end
```



```
integer ret;
begin
ret = $sscanf(str,"%s_%d",name,no);
end
```



Or actually use the return value as it was intended

```
begin
if ($fseek(fd,0,0) == -1)
	$display("ERROR: fseek failed");
end
```



