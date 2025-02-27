---
title: Персистентность
weight: 11
authors:
- Сергей Слотин
date: 2021-09-11
---

Прсистентные структуры данных (англ. *persistent data structures*) — это структуры данных, которые при внесении в них изменений сохраняют доступ ко всем своим предыдущим состояниям.

Есть несколько «уровней» персистентности:

- Частичная — к каждой версии можно делать запросы, но изменять можно только последнюю.
- Полная — можно делать запросы к любой версии и менять любую версию.
- Конфлюэнтная — помимо этого можно объединять две структуры данных в одну (например, сливать вместе кучи или деревья поиска).
- Функциональная — структуру можно реализовать на чистом функциональном языке: для любой переменной значение может быть присвоено только один раз и изменять значения переменных нельзя.

Персистентные структуры используются в текстовых редакторах, системах контроля версий, базах данных, в некоторых параллельных алгоритмах, а также в функциональном программировании.

Помимо этого, персистентные структуры нередко используются и для решения «обычных» алгоритмических задач, что и будет основным фокусом этой главы.
