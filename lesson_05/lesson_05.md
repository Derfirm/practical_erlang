## Key-Value структуры данных

Было бы трудно пользоваться языком, в котором нет key-value структур
данных.  И в эрланг они есть. И поскольку они очень востребованы в
любом проекте, рассмотрим их подробнее.


## proplists

Самое простое, что можно придумать, это собрать пару "ключ-значение" в
кортеж, и положить такие кортежи в список.

```erlang
1> PropList = [{key1, "Val1"}, {key2, 2}, {key3, true}].
[{key1,"Val1"},{key2,2},{key3,true}]
```

Именно это и делает модуль [proplists](http://www.erlang.org/doc/man/proplists.html).

Только proplists еще позволяет пары, где значение **true** сокращать,
сохраняя вместо кортежа просто ключ.

```erlang
2> PropList2 = [{key1, "Val1"}, {key2, 2}, key3].
[{key1,"Val1"},{key2,2},key3]
```

АПИ модуля довольно странное. Есть несколько функций для извлечения
значения по ключу, есть функция для удаление значения. Но нет функций
для добавления и изменения значения.

Впрочем, с добавлением все просто. Для этого используем оператор **cons**:

```erlang
3> PropList3 = [{key4, "Hello"} | PropList2].
[{key4,"Hello"},{key1,"Val1"},{key2,2},key3]
```

С изменением значения тоже просто, для этого опять используем оператор **cons** :)

```erlang
4> PropList4 = [{key1, "New val"} | PropList3].
[{key1,"New val"},
 {key4,"Hello"},
 {key1,"Val1"},
 {key2,2},
 key3]
```

Тогда оказывается, что в списке есть два ключа **key1**, но proplists
такое разрешает. В этом случае будет возвращаться первое от головы
списка значение.

Для извлечения значения по ключу есть функции **get\_value/2**,
**get\_value/3** и **get\_all\_values/2**.

```erlang
5> proplists:get_value(key1, PropList4).
"New val"
6> proplists:get_value(key777, PropList4).
undefined
7> proplists:get_value(key777, PropList4, "default value").
"default value"
8> proplists:get_all_values(key1, PropList4).
["New val","Val1"]
9> proplists:get_all_values(key777, PropList4).
[]
```

Есть еще функции **lookup/2** и **lookup_all/2**, они отличаются тем,
что возвращают не значение, а кортеж ключ-значение.

```erlang
10> proplists:lookup(key1, PropList4).
{key1,"New val"}
11> proplists:lookup(key777, PropList4).
none
12> proplists:lookup_all(key1, PropList4).
[{key1,"New val"},{key1,"Val1"}]
13> proplists:lookup_all(key777, PropList4).
[]
```

Ну и функция **delete/2** удаляет значение из списка:

```erlang
14> proplists:delete(key1, PropList4).
[{key4,"Hello"},{key2,2},key3]
```

Понятно, что такая структура данных не очень эффективна.  Операции
поиска и удаления выполняются, конечно, не за логарифмическое время, а
за линейное. Несмотря на это, proplists популярен, и широко
используется в проектах. Обычно он используется для конфигурирования,
для хранения различных настроек и опций.

Ну и в других случаях, когда мы знаем, что ключей в наших данных будет
не много, не больше нескольких десятков, то смело берем proplists.
Ибо в этой ситуации его эффективность не важна.


## dict

dicts are proplists with a taste for formality.

http://www.erlang.org/doc/man/dict.html

new
append, store
append\_list то есть, у одного ключа может быть много значений
erase
fetch, find
map, fold
from\_list, to\_list
merge

TODO пример CRUD операций

dict:find/2 (when you do not know whether the key is in the dictionaries),
dict:fetch/2 (when you know it is there or that it must be there)

1> D = dict:new().
2> D2 = dict:append(1, "Bob", D).
3> D3 = dict:append(2, "Bill", D2).
4> dict:find(1, D3).
{ok,["Bob"]}
5> dict:find(2, D3).
{ok,["Bill"]}
6> dict:find(3, D3).
error
8> D4 = dict:append(1, "John", D3).
9> dict:find(1, D4).
{ok,["Bob","John"]}
10> D5 = dict:erase(1, D4).
11> dict:find(1, D5).
error
12> dict:to_list(D5).
[{2,["Bill"]}]
13> dict:fetch(2, D5).
["Bill"]
14> dict:fetch(1, D5).
    exception error: bad argument


## orddict

Orddict implements a Key - Value dictionary. An orddict is a
representation of a dictionary, where a list of pairs is used to store
the keys and values. The list is ordered after the keys.

This module provides exactly the same interface as the module dict but with a defined representation.

http://www.erlang.org/doc/man/orddict.html


Orddicts are a generally good compromise between complexity and
efficiency up to about 75 elements (see my benchmark). After that
amount, you should switch to different key-value stores.

There are basically two key-value structures/modules to deal with larger amounts of data: dicts and gb_trees.

Ну странно получается, 75 элементов -- мелочь. Ради них и не требуется сортировка.
Фред пишет, что dict эффективнее на большем к-ве элементов. Тогда нафига вообще нужен orddict?

TODO погонять самому бенчмарк Фреда
http://learnyousomeerlang.com/static/erlang/keyval_benchmark.erl


## gb_trees

http://www.erlang.org/doc/man/gb_trees.html

General Balanced Trees, on the other hand, have a bunch more functions
leaving you more direct control over how the structure is to be used.

There are basically two modes for gb\_trees: the mode where you know
your structure in and out (I call this the 'smart mode'), and the mode
where you can't assume much about it (I call this one the 'naive
mode').

In naive mode, the functions are
gb\_trees:enter/3
gb\_trees:lookup/2
gb\_trees:delete_any/2.

The related smart functions are
gb\_trees:insert/3
gb\_trees:get/2
gb\_trees:update/3
gb\_trees:delete/2.

The disadvantage of 'naive' functions over 'smart' ones is that
because gb_trees are balanced trees, whenever you insert a new element
(or delete a bunch), it might be possible that the tree will need to
balance itself. This can take time and memory (even in useless checks
just to make sure). The 'smart' function all assume that the key is
present in the tree: this lets you skip all the safety checks and
results in faster times.

То есть, если мы предполагаем, что после изменений балансировать
дерево не нужно, то пользуемся smart функциями. А если нужно,
то пользуемся naive функциями. Но тогда при чем тут функции
чтения lookup и get?

Как насчет такого варианта:
нужно сделать несколько вставок. Мы не хотим перебалансировать
после каждой вставки, а хотим один раз, после всех вставок.
Можно сделать все ставки smart, и последнюю naive?
TODO: это нужно как-то выяснить

Oh and also note that while dicts have a fold function, gb_trees don't: they instead have an iterator function
There is also gb\_trees:map/2, which is always a nice thing when you need it.


(general balanced trees)

TODO пример CRUD операций
enter, insert, update
get, lookup
delete, delete\_any

TODO это нужно проверить
gb_trees хранит ключи, понятно, в дереве. Поэтому еще более быстрый поиск O(ln(n)), но добавление может оказаться медленным, из-за необходимости перебалансировать дерево.

deletion only does not rebalance the tree
balance/1 после многих удалений

## maps

Все выше -- реализации средствами языка, то есть, поверх списков и структур данных

Maps -- нативная реализация в виртуальной машине.

http://www.erlang.org/doc/man/maps.html

TODO прочитать Фреда
http://learnyousomeerlang.com/maps

TODO пример CRUD операций

new
put, update
find, get
remove

merge
fold, map

from\_list, to\_list

TODO разобраться, какой сахар работает, а какой задуман, но еще не работает.

Maps are associative collections of key-value pairs. The key can be any Erlang
term. In Perl and Ruby they are called hashes; in C++ and Java they are called
maps, in Lua they are called tables, and in Python they are called dictionaries.

maps to\_json, from\_json -- во как.


## Заключение

I would therefore say that your application's needs is what should
govern which key-value store to choose. Different factors such as how
much data you've got to store, what you need to do with it and whatnot
all have their importance. Measure, profile and benchmark to make
sure.

ets на следующем уроке