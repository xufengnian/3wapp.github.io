# Grep

## 单引号 vs 双引号

一般常量用单引号''括起，如果含有变量则用双引号""括起。

最大不同：单引号与双引号的最大不同在于双引号仍然可以保有变数的内容，但单引号内仅能是一般字元，而不会有特殊符号.

使用举例：

* “”号里面遇到`$`，`\` 等特殊字符会进行相应的变量替换
* ‘’号里面的所有字符都保持原样

对于字符串，两者相同，匹配模式也大致相同

## 参数

```
-A num, --after-context=num       在结果中同时输出匹配行之后的num行
-B num, --before-context=num      在结果中同时输出匹配行之前的num行，有时候我们需要显示几行上下文。
-i, --ignore-case                 忽略大小写
-n, --line-number                 显示行号
-R, -r, --recursive               递归搜索子目录
-v, --invert-match                输出没有匹配的行
```

## 样例


```
grep -nri ctf{ /home/*
```
