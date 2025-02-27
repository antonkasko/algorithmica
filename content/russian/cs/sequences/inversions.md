---
# TODO: реализация
title: Число инверсий
weight: 5
authors:
- Сергей Слотин
---

Пусть у нас есть некоторая перестановка $p$ (какая-то последовательность чисел от $1$ до $n$, где все числа встречаются ровно один раз). *Инверсией* называется пара индексов $i$ и $j$ такая, что $i < j$ и $p_i > p_j$. Требуется найти количество инверсий в данной перестановке.

## Наивный алгоритм

Эта задача легко решается за $O(n^2)$ обычным перебором всех пар индексов и проверкой каждого на инверсию:

```cpp
int count_inversions(int *p, int n) {
    int res = 0;
    for (int i = 0; i < n; i++)
        for (int j = i + 1; j < n; j++)
            if (p[i] > p[j])
                res++;
    return res;
}
```

## Сортировкой слиянием

Внезапно эту задачу можно решить сортировкой слиянием, слегка модифицировав её.

Оставим ту же идею: пусть мы умеем находить количество инверсий в массиве размера $n$, научимся находить количество инверсий в массиве размера $2n$.

Если мы запустимся рекурсивно от половин, то мы уже узнаем количество инверсий в левой половине и в правой половине массива по отдельности. Осталось лишь посчитать число инверсий, где одно число лежит в левой половине, а второе в правой половине.

Давайте подробнее рассмотрим операцию слияния левой и правой половины. Первый указатель указывает на элемент левой половины, второй указатель указывает на элемент второй половины, мы смотрим на меньший из них и этот указатель вдигаем вправо.

Рассмотрим число $x$ в левой половине. В скольких инверсиях между половинами оно участвует? В количестве, равному количеству чисел в правой половине меньше, чем оно. Знаем ли мы это количество? Да! Мы его узнаем его ровно в тот момент, когда мы число $x$ записываем в слитый массив — второй указатель указывает на первое число в правой половине, которое больше $x$. Значит в тот момент, когда мы добавляем число $A$ из левой половины, к ответу достаточно прибавить индекс второго указателя относительно начала правой половины. Так мы учтем все инверсии между половинами.

## Структурами для суммы на отрезке

Подумаем над определением инверсии: это такая пара индексов $i$ и $j$, что $i < j$ и $p_i > p_j$.

Нанесем индексы и их $p_i$ на двумерную плоскость. Для фиксированной точки $(i, p_i)$, как выглядит наше условие на $(j, p_j)$ геометрически?

Это какой-то прямоугольник, в который включены все точки, у которых первая координата больше, а вторая меньше.

Значит, мы можем взять [какую-нибудь структуру](/cs/range-queries/segment-tree) для единичного прибавления и суммы на префиксе и применить метод сканирующей прямой, пройдясь по всем элементам $i$ начиная с конца, и будем прибавлять единицу в ячейку $p_i$ и находить все подходящие $j$ суммой на префиксе $p_i$ — у всех учтенных точек $p_j$ будет меньше, потому что этот префикс, а $j$ будет больше, потому что мы проходим справа налево — и прибавлять её к ответу.

> Посчитать число беспорядков в перестановке из $n$ элементов (беспорядок или инверсия — это пара чисел $i < j$, для которых $p_i > p_j$).

Эта задача решается просто, если уметь писать сортировку слиянием вручную. Но мы пойдем по другому пути. Создадим ДО для суммы на $n$ элементов, изначально заполненное нулями. Теперь будем проходить по этому массиву слева направо. Когда обрабатываем очередное число $x$, будем делать две вещи:

* Запросим сумму от $k$ до $n$ в ДО.
* Добавим единичку в $k$-тую позицию в ДО.

Так мы для каждой инверсии учтём её, когда запросим сумму для её правого элемента. Таким образом, мы решили эту задачу за $O(n \log n)$ запросов.

