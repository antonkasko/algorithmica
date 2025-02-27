---
title: Корневые эвристики
authors:
- Сергей Слотин
- Иван Сафонов
weight: 3
---

Корневые эвристики — это обобщённое название различных методов и структур данных, опирающихся на тот факт, что если мы разделим какое-то множество из $n$ элементов на блоки по $\sqrt{n}$ элементов, то самих этих блоков будет не более $\sqrt{n}$.

Центральное равенство этой статьи: $\sqrt x = \frac{x}{\sqrt x}$.

## Деление на тяжелые и легкие объекты

Всем известный алгоритм факторизации за корень опирается на тот факт, что каждому «большому» делителю $d \geq \sqrt n$ числа $n$ соответствует какой-то «маленький» делитель $\frac{n}{d} \leq n$.

Подобное полезное свойство (что маленькие объекты маленькие, а больших объектов не много) можно найти и у других объектов.

### Длинные и короткие строки

[Задача](https://codeforces.com/contest/710/problem/F). Требутся в онлайне обрабатывать три типа операций над множеством строк:

1. Добавить строку в множество.
2. Удалить строку из множества.
3. Для заданной строки, найти количество её вхождений как подстроку среди всех строк множества.

Одно из решений следующее: разделим все строки на *короткие* ($|s| < \sqrt l$) и *длинные* ($|s| \geq \sqrt l$), где $l$ означает суммарную длину всех строк. Заметим, что длинных строк немного — не более $\sqrt l$.

С запросами будем справляться так:

- Заведём хэш-таблицу, и когда будем обрабатывать запрос добавления или удаления, будем прибавлять или отнимать соответственно единицу по [хэшам](/cs/hashing) всех её коротких подстрок. Это можно сделать суммарно за $O(l \sqrt l)$: для каждой строки нужно перебрать $O(\sqrt l)$ разных длин и окном пройтись по всей строке.
- Для запроса третьего типа для короткой строки, просто посчитаем её хэш и посмотрим на значение в хэш-таблице.
- Для запроса третьего типа для длинной строки, мы можем позволить себе посмотреть на все неудалённые строки, потому что таких случаев будет немного, и если мы можем за линейное время найти все вхождения новой строки, то работать это будет тоже за $O(l \sqrt l)$. Например, можно посчитать [z-функцию](/cs/string-searching/z-function) для всех строк вида `s#t`, где $s$ это строка из запроса, а $t$ это строка из множества; здесь, правда, есть нюанс: $s$ может быть большой, а маленьких строк $t$ много — нужно посчитать z-функцию сначала только от $s$, а затем виртуально дописывать к ней каждую $t$ и досчитывать функцию.

Иногда отдельный подход к тяжелым и лёгким объектам не требуется, но сама идея помогает увидеть, что некоторые простые решения работают быстрее, чем кажется.

### Треугольники в графе

**Задача.** Дан граф из $n$ вершин и $m \approx n$ рёбер. Требуется найти в нём количество циклов длины три.

Будем называть вершину *тяжелой*, если она соединена с более чем $\sqrt n$ другими вершинами, и *лёгкой* в противном случае.

Попытаемся оценить количество соединённых вместе троек вершин, рассмотрев все возможные 4 варианта:

0. В цикле нет тяжелых вершин. Рассмотрим какое-нибудь ребро $(a, b)$ цикла. Третья вершина $c$ должна лежать в объединении списков смежности $g_a$ и $g_b$, а раз обе эти вершины лёгкие, то таких вершин найдётся не более $\sqrt n$. Значит, всего циклов этого типа может быть не более $O(m \sqrt n)$.
1. В цикле одна тяжелая вершина. Аналогично — есть одно «лёгкое» ребро, а значит таких циклов тоже $O(m \sqrt n)$.
2. В цикле две тяжелые вершины — обозначим их как $a$ и $b$, а лёгкую как $c$. Зафиксируем пару $(a, c)$ — способов это сделать $O(m)$, потому что всего столько рёбер. Для этого ребра будет не более $O(\sqrt n)$ рёбер $(a, b)$, потому что столько всего тяжелых вершин. Получается, что всего таких циклов может быть не более $O(m \sqrt n)$.
3. Все вершины тяжелые. Аналогично — тип третьей вершины в разборе предыдущего случая нигде не использовался; важно лишь то, что тяжелых вершин $b$ немного.

Получается, что циклов длины 3 в графе может быть не так уж и много — не более $O(m \sqrt n)$.

Само решение максимально простое: отсортируем вершины графа по их степени, ориентируем ребра $v \rightarrow u, v \le u$; теперь внутренним циклом будем перебирать пути $v \rightarrow u \rightarrow w, v \le u \le w$, а потом проверять существование ребра $v \rightarrow w$.

```c++
vector<int> g[maxn], p(n); // исходный граф и список номеров вершин
iota(p.begin(), p.end(), 0); // 0, 1, 2, 3, ...

// чтобы не копипастить сравнение:
auto cmp = [&](int a, int b) {
    return g[a].size() < g[b].size() || (g[a].size() == g[b].size() && a < b);
};

// в таком порядке мы будем перебирать вершины
sort(p.begin(), p.end(), cmp);

// теперь удалим все лишние рёбра (ведущие в более тяжелые вершины)
for (int v = 0; v < n; v++) {
    vector<int> &t = g[v];
    // отсортируем их и удалим какой-то суффикс
    sort(t.begin(), t.end(), cmp);
    while (t.size() > 0 && cmp(t.back(), v))
        t.pop_back();
    reverse(t.begin(), t.end());
}

// рядом с каждой вершиной будем хранить количество
// ранее просмотренных входящих рёбер (v -> w)
vector<int> cnt(n, 0);
int ans = 0;

for (int v : p) {
    for (int w : g[v])
        cnt[w]++;
    for (int u : g[v])
        for (int w : g[u])
            ans += cnt[w]; // если в графе нет петель, то cnt[w] это 0 или 1
    // при переходе к следующему v массив нужно занулить обратно
    for (int w : g[v])
        cnt[w]--;
}
```

Задачу также можно было решить чуть более прямолинейно, переместив все проверки внутрь основного цикла, но это сказалось бы на скорости работы.

### Рюкзак за $O(S \sqrt S)$

Если у нас есть $n$ предметов с весами $w_1$, $w_2$, $\ldots$, $w_n$, такими что $\sum w_i = S$, то мы можем решить задачу о рюкзаке за время $O(S \cdot n)$ стандартной динамикой. Чтобы решить задачу быстрее, попытаемся сделать так, чтобы число предметов стало $O(\sqrt S)$.

Заметим, что количество различных весов будет $O(\sqrt S)$, так как для любых $k$ различных чисел с суммой $S$

$$
S = w_1 + w_2 + \ldots + w_n \geq 1 + 2 + \ldots + k = \frac{k \cdot (k+1)}{2}
$$

Откуда значит, что $k \leq 2\sqrt S = O(\sqrt S)$.

Рассмотрим теперь некоторый вес $x$, который $k$ раз встречается в наборе весов. «Разложим» $k$ по степеням двойки и вместо всех $k$ вхождений этого веса добавим веса $x$, $2 \cdot x$, $4 \cdot x$, $\ldots$, $(k - 1 - 2^t) \cdot x$, где $t$ это максимальное целое число, для которого выполняется $2^t − 1 \leq k$. Легко видеть, что все суммы вида $q \cdot x$ ($q \leq k$) и только их по-прежнему можно набрать.

Алгоритм в этом, собственно, и заключается: проведем данную операцию со всеми уникальными значениями весов и после чего запустим стандартное решение. Уже сейчас легко видеть, что новое количество предметов будет $O(\sqrt S \log S)$, потому что для каждого веса мы оставили не более $\log S$ весов, а всего различных весов было $O(\sqrt S)$.

**Упражнение.** Докажите, что предметов на самом деле будет $O(\sqrt S)$.

*Примечание.* Классическое решение рюкзака можно ускорить на несколько порядков, если использовать [bitset](/cs/set-structures/bitset).

---

Также корневые эвристики можно использовать в контексте [структур данных](/cs/range-queries/sqrt-structures) и [обработки запросов](../mo).
