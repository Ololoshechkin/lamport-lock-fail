# Ошибочный алгоритм Лампорта

[![Build Status](https://travis-ci.com/ITMO-MPP-2018/lamport-lock-fail-Ololoshechkin.svg?token=B2yLGFz6qwxKVjbLm9Ak&branch=master)](https://travis-ci.com/ITMO-MPP-2018/lamport-lock-fail-Ololoshechkin)

В рамках данного задания вы проанализируете некорректную попытку реализации алгоритма Лампорта для взаимного исключения.

Вася Пупкин увидел второй вариант реализации алгоритма Лампора (см. лекцию), очень порадовался его простоте и решил его соптимизировать. Он избавился от массива `choosing` и получил вот такой код, явно расписав процедуру выбора
максимальной метки в отдельный цикл, используя значение *0* для обозначения
отсутсвия интереса потока к взятию блокировки:

```java
threadlocal int id       // 0..N-1 -- идентификатор потока
shared      int label[N] // заполненно нулями по умолчанию

def lock:
  1: my = 1 // номер билета текущего потока
  2: for k in range(N): if k != id:
  3:     my = max(my, label[k] + 1) // должен быть больше, чем у других
  4: label[id] = my // публикуем свой номер билета для других потоков
  5: for k in range(N): if k != id:
  6:     while true: // пропускаем поток k до тех пока, пока номер его билета меньше
  7:         other = label[k] // читаем номер билета потока k
  8:         if other == 0 or (other, k) > (my, id): break@6 // если номер его билета меньше, перестаем ждать  

def unlock:
  9: label[id] = 0
```

Чтения и записи элементов разделяемого массива `label` происходят атомарно.

Обратите внимание, что Вася ожидал подвох с чтениями элементов массива `label`, поэтому на строке 7 вставил отдельное чтение из массива `label` в локальную переменную `other`, чтобы дальше проанализировать его значение не проводя несколько отдельных операций чтения (иначе разные чтения могут прочитать разные значения).

## Задание

1. Продемонстрируйте испольнение данного алгоритма на двух потоках
(`N == 2`), которое нарушает условие взаимного исключения.

 Описание исполнения должно содержаться в файле [execution](execution) и содержать набор событий (одно событие = одна строка) в следующем формате:
 ```
 <tid> <line> <action> <location> <value>
 ```
 * `<tid>` - id потока, `0` или `1`;
 * `<line>` - номер строчки кода, которая исполняется; `3`, `4` или `7`;
 * `<action>` - тип события, `rd` (чтение) или `wr` (запись)
 * `<location>` - описание разделяемой переменной, к которой происходит обращение, `label[<index>]`;
 * `<value>` - значение, которое записано или прочитано; например, `10`.

 Для проверки корректности вашего примера, запустите `./gradlew run` из корня репозитория. Описание должно содержаться в файле [Solution.md](Solution.md).

 Если у вас не получается найти решение силой мысли, то проанализуйте в черновике варианты одновременного исполнения двумя потоками операции `lock` методом чередования операций. Обратите внимание, что при анализе вас интересуют только те строки кода, которые обращаются к разделяемым переменным. В данном слуаче это обращения к элементам массива `label` (строки 3, 4, 7).

2. Опишите как можно было бы исправить этот код, не заводя отдельного
массива `choosing`, но использую концептуально похожую идею.

## Формат сдачи

Выполняйте задание в этом репозитории. По готовности добавьте "+" в таблицу с оценками в столбец "Готово" текущего задания. 

В случае необходимости доработки домашнего задания после проверки, "+" в таблице замененяется на "?" и создается issue на GitHub-е. Как только необходимые исправления произведены, заменяйте "?" обратно на "+" и закрывайте issue. После этого задание будет проверено ещё раз.

Перед сдачей задания замените `<your_GitHub_account>` в начале данного файла на свой логин в GitHub для получения информации о сборке в Travis. Это нужно сделать в двух местах: ссылка на картинку и на билд в Travis-е.



