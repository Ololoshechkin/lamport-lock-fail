## 1. Некорректное исполнение
[![Build Status](https://travis-ci.com/ITMO-MPP-2017/lamport-lock-fail-<your_GitHub_account>.svg?token=B2yLGFz6qwxKVjbLm9Ak&branch=master)](https://travis-ci.com/ITMO-MPP-2017/lamport-lock-fail-<your_GitHub_account>)


## 2. Исправление алгоритма
Например, мы можем понимать что какой-то поток делает choosing, выставив его label в -1 и ждать пока этот label станет не -1:

```java
threadlocal int id       // 0..N-1 -- идентификатор потока
shared      int label[N] // заполненно нулями по умолчанию

def lock:
  1: my = 1 // номер билета текущего потока
  2: label[id] = -1 // мы choosing
  3: for k in range(N): if k != id:
  4:     my = max(my, label[k] + 1) // должен быть больше, чем у других
  5: label[id] = my // публикуем свой номер билета для других потоков
  6: label[id] = -1 // мы больше не choosing
  7: for k in range(N): if k != id:
  8:     while true: // пропускаем поток k до тех пока, пока номер его билета меньше
  9:         other = label[k] // читаем номер билета потока k
  10:         if other == -1 continue@6
  11:         if other == 0 or (other, k) > (my, id): break@6 // если номер его билета меньше, перестаем ждать  

def unlock:
  9: label[id] = 0
```