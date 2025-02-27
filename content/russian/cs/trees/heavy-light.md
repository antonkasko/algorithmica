---
title: Heavy-light декомпозиция
authors:
- Сергей Слотин
- Александр Кульков
weight: 7
created: 2019
date: 2021-08-27
---

Heavy-light декомпозиция — это мощный метод решения задач на запросы на путях, когда также есть запросы обновлений. Если запросов обновления нет, то лучше написать [что-нибудь попроще](../centroid).

Heavy-light декомпозицией корневого дерева называется результат следующего процесса: каждой вершины $v$ посмотрим на всех её непосредственных детей $u$, выберем среди них ребёнка $u_{max}$ с самым большим размером поддерева — если таких больше одного, то любого из них — и назовём ребро $(v, u_{max})$ тяжелым (англ. heavy), а все остальные рёбра — лёгкими (англ. light).

Таким образом, дерево распалось на тяжелые и лёгкие рёбра. Это разбиение и называют heavy-light декомпозицией.

![](../img/heavy-light.png)

Докажем несколько полезных свойств такого разбиения.

**Утверждение.** Дерево разбивается на непересекающиеся пути из тяжелых рёбер.

**Доказательство.** В каждую вершину входит не более одного тяжелого ребра, и из каждой вершины исходит не более одного тяжелого ребра.

Назовём *блоком* либо лёгкое ребро, либо вертикальный путь из тяжелых рёбер.

**Утверждение.** На любом вертикальном пути будет не более $O(\log n)$ блоков.

**Доказательство** разбивается на две части:

* Лёгких ребер на вертикальном пути будет не более $O(\log n)$: рассмотрим самую нижнюю вершину и будем идти по вертикальному пути снизу вверх. Каждый раз, когда мы переходим по лёгкому ребру, размер поддерева текущей вершины увеличивается в два раза, потому что если вершина связана с родителем лёгким ребром, то у него есть какой-то другой ребёнок, который не легче текущего.
* Непрерывных путей из тяжелых рёбер будет не более $O(\log n$: если это не конец или начало пути, то каждый такой путь окружают два лёгких ребра, а их всего $O(\log n)$.

**Следствие.** На любом пути будет не более $O(\log n)$ блоков.

Ради этого мы всё и делали: теперь построим какую-нибудь структуру на каждом тяжелого пути (например, [дерево отрезков](/cs/segment-tree)), а при ответе на запрос (скажем, суммы на пути) разобьем его на $O(\log n)$ запросов либо к подотрезкам тяжелых путей, либо к лёгким рёбрам.

## Реализация

Большинство публичных реализаций HLD — это 120-150 строк кода. Мы же воспользуемся следующим трюком, который сильно упростит нам жизнь: перенумеруем вершины дерева так, что для каждого тяжелого пути все его вершины будут иметь последовательные номера.

А именно, на этапе подсчёта размера поддеревьев, изменим список смежности каждой вершины так, чтобы в самом начале шел её «тяжелый» ребёнок. Тогда, если запустить обычный эйлеров обход графа, то массив `tin` будет нужной нумерацией, потому что в каждой вершине мы шли в своего тяжелого ребёнка в первую очередь.

```c++
vector<int> g[maxn];
int s[maxn], p[maxn], tin[maxn], tout[maxn];
int head[maxn]; // «голова» тяжелого пути, которому принадлежит v
int t = 0;

void sizes(int v = 0) {
    s[v] = 1;
    for (int &u : g[v]) {
        sizes(u);
        s[v] += s[u];
        if (s[u] > s[g[v][0]])
            // &u -- это ссылка, так что её легально использовать при swap-е
            swap(u, g[v][0]);
    }
}

void hld(int v = 0) {
    tin[v] = t++;
    for (int u : g[v]) {
        // если это тяжелый ребенок -- его next нужно передать
        // в противном случае он сам является головой нового пути
        head[u] = (u == g[v][0] ? head[v] : u);
        hld(u);
    }
    tout[v] = t;
}
```

Теперь мы можем построить дерево отрезков или какую-нибудь другую структуру поверх массива размера $n$ и при запросе к какому-нибудь тяжелому пути делать запрос к отрезку в этой структуре.

## Как им решать задачи

Простейший пример задачи на HLD: дано дерево, в каждой вершине которого записано какое-то число, и поступают запросы двух типов:

1. Узнать минимальное число на пути между $v_i$ и $u_i$.
2. Заменить число $v_i$-той вершины на $x_i$.

Подвесим дерево за произвольную вершину и построим на нём HL-декомпозицию с деревом отрезков в качестве внутренней структуры. Его код мы приводить не будем и посчитаем, что оно реализовано примерно так же, как в [соответствующей статье](/cs/segment-tree) и имеет методы `upd(k, x)` и `get_min(l, r)`.

```c++
int val[maxn];
segtree st(0, n);
```

При операции обновления нам нужно просто обновить нужную ячейку в дереве отрезков:

```c++
void upd(int v, int x) {
    st.upd(tin[v], x);
}
```

Запрос минимума сложнее — нам нужно разбить исходный запрос на запросы к вертикальным путям:

```c++
int ancestor(int a, int b) {
    return tin[a] <= tin[b] && tin[b] < tout[a];
}

void up(int &a, int &b, int &ans) {
    while (!ancestor(head[a], b)) {
        ans = min(ans, st.get_min(tin[head[a]], tin[a]));
        a = p[head[a]];
    }
}

int get_min(int a, int b) {
    int ans = inf;
    up(a, b, ans);
    up(b, a, ans);
    if (!ancestor(a, b))
        swap(a, b);
    ans = min(ans, st.get_min(tin[a], tin[b]));
    return ans;
}
```

Заметьте, что чтобы разбить путь на два вертикальных, нам даже не нужно отдельно решать задачу LCA: поднимаясь по тяжелым путям, мы за $O(\log n)$ подъемов приходим к наибольшему общему предку.

## Замечания

- Если в задаче нет запросов обновления, то можно «кэшировать» запросы. Заметим, что большинство запросов к структуре, которые надо сделать, находятся на префиксах тяжелых путей, и на них можно отвечать за $O(1)$ предпосчетом, и префиксных подзапросов будет $O(\log n)$ на запрос. Также придется сделать $O(1)$ запросов к структуре, которые будут работать за $O(\log n)$. Получаем $O(\log n)$ на запрос.
- Так как наша реализация HLD строит структуру данных на эйлеровом обходе дерева, мы можем добавить запросы к поддеревьям, ведь поддеревья — это подотрезки эйлерова обхода.
