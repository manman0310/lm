---
layout: blog_default
title:  "shell脚本爬坑杂记"
---

# shell脚本爬坑杂记

在此记录淌过shell脚本的坑

### 0. 处理shell中的10个以上参数

传入参数时，  
传入脚本的第10个参数如果用$10接收，则会被替代为第1个参数+0；  
第11个参数如果用$11接收，则会被替代为第1个参数+1……

**正确写法**

``` sh
${10}  
${11}

```

### 1. shell exit码

exit 退出状态只能是一个介于 0~255 之间的整数。
其中只有 0 表示成功，其它值都表示失败。

### 2. 判断字符串中特定字符方法   

``` shell
  preStr='asdfsdf'
  if [[ $preStr == *as* ]]; then
        echo "with as"
    else
        echo "without as"
    fi
```
