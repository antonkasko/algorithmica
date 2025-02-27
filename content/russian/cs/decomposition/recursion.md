---
title: Рекурсия
draft: true
---

На самом деле каждый раз, когда вы используете рекурсию, вы используете
стек, ведь она реализована с его помощью.

А именно, представьте, что вы вызываете функцию f(n), и внутри нее
вызывается другая функция g(n). Но ведь когда g(n) перестанет
выполняться, вам нужно продолжить выполнять f(n). А значит все это
время, пока вы вычисляете g(n), вам нужно хранить информацию о всех
аргументах и локальных переменных внутри функции f(n), и даже место,
где вы там остановились. Удобно хранить всю эту информацию в стеке -
когда вы вызываете функцию, все содержимое прошлой функции
сохраняется в стек, а когда функция перестает выполняться -
прошлая функция снимает со стека последнюю функцию, и продолжает ее
выполнять.

Именно поэтому все задачи на рекурсию можно решить и без рекурсии, и
возможно это даже будет быстрее. Достаточно лишь написать while,
который вынимает функцию вместе со всей информацией про ее
выполенени с верха стека, выполняет её до следующего шага, где
нужно вызвать другую функцию, кладет на верх стека обновленную функцию и
новую вызванную функцию. И так, пока стек не станет пустым.

Стек рекурсии ограничен в некоторых языках прогрмамирования (например,
питон), в C++ стек рекурсии ограничен лишь вашей оперативной памятью.

Пример задачи на рекурсию, которую теоретически можно переписать под
стек - это Ханойские башни.

## Условие.

У вас есть три стержня (= стека), на первом находятся $N$ дисков, причем
чем диск ниже, тем он шире. Можно переносить верхний диска одного
стержня на верх другого стержня, если только под ним не будет
находиться меньший по ширине диск. Нужно вывести последовательность
операций, после которой все диски перенесутся на третий стержень.

## Решение.

Достаточно делать как в динамическом программировании - формализовать
задачу и свести ее к предыдущим. Давайте функция hanoi(n, from, to,
middle) будет выводить все операции с переносом $n$ стержней с верха
стержня $from$ на стержень $to$, при условии, что их можно еще
пользоваться стержнем $middle$ (и при этом на стержнях $to$ и
$middle$ все диски шире, чем эти $n$, чтобы их можно было класть
сверху). Тогда на самом деле эта операция разбивается на такие:

  - hanoi(n - 1, from, middle, to) - перенести $n-1$ диск на средний
    стержень
  - print(from + " -\> " + to) - перенести самый широкий диск на
    финальный стержень
  - hanoi(n - 1, middle, to, from) - перенести $n-1$ диск со среднего
    стержаня на финальный

Алгоритм работает за $O(2^n)$, так как вызывает два таких же алгоритма
для $n-1$.