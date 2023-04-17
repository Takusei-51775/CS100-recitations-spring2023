---
marp: true
math: mathjax
---

---

# HW1-1

---

# HW1-2


```c
scanf(" (%d,%d)", &x1, &y1);
scanf(" (%d,%d)", &x2, &y2);
```
当输入是
(0, 1)
(1, 2)
的时候，电脑看到的是：
`(0, 1)'\n'(1, 2)'\n'`
`%d`会舍弃前面的空白字符（`' '`, `'\n'`, `'\r'`)

---

# HW1-2


```c
char line[10];
fgets(line, 10, stdin);
```
LF(Linux)与CRLF(Windows)
敲回车，输入的可能是`\n`，也可能是`\r\n`!

---

# HW1-3

## Don't Repeat Yourself!

```c
if (/* ... */) {
  eat();
  sleep();
  work();
} 
else {
  eat();
  sleep();
  play();
}
```

---

# HW1-3

## Don't Repeat Yourself!

```c

eat();
sleep();
if (/* ... */) {
  work();
} 
else {
  play();
}
```

---

# HW1-3

## Don't Repeat Yourself!

```c
if (/* ... */) {
  eat();
  sleep();
  work();
} 
else {
  if (/* ... */){
    doSomethingElse();
  }
  else {
    eat();
    sleep();
    play();
  }
}
```

---

# HW1-3


不能提取出来？写成函数！
```c
void doSameThings() {
  eat();
  sleep();
}

if (/* ... */) {
  doSameThings();
  work();
} 
else {
  if (/* ... */){
    doSomethingElse();
  }
  else {
    doSameThings();
    play();
  }
}
```

---