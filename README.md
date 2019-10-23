# Temporal

Storage of temporal data in Caché.


## Installation

- Download the Temporal package and compile it

## Intro

Each Property is stored in a separate node:  `Package.ClassD.PropertyD(id, timestamp) = val` where:

- `Package.ClassD` - class data global
- `Package.ClassD.PropertyD` - global Property property
  + `id` is the primary key
  + `timestamp` - date/time in unixtime format
  + `val` - value of the property (For the data type, we write the value, for the data type - id, serialize the serialized ones, wrap the stream into classes and write the id, serialize the lists/arrays of the objects into lists/arrays id, and then serialize the lists/arrays of the data)

Also the node: `Package.ClassD.PropertyD(id) = val` with the most recent value of the property is automatically generated.

## Property Getter

### Object context

Load into memory happens on the first call, then we take it from memory. The algorithm is as follows:

1. If the value is in memory, we return it. EXIT
2. If we need the latest version of the data, we'll take it.
3. If we need the data at the moment %ts, we try to take the data at the moment `%ts`
4. If we have not found 3, we take the following/previous value
5. If we need a value for the date `>%ts`, but it does not exist, then take the previous value (should it be the same as 2? TODO)
6. Write the received value into the memory
7. Return the value

### SQL context

For SQL everything is similar, but only steps 2-5, 7 are executed.

## Limitations

- No INSERT/UPDATE - you can write a trigger
- The problem of simultaneous updates is not solved - Lock was invented just for this purpose
- No plain indices for current values - I don't see how to solve it without func indices, which are needed anyway
- Temporary variables `%ts`/`%state` when used in COS SQL should be either new before or deleted after
- No storage of object change metadata - Need to? Use case? How? Project to class {class, id, ts, diff}. In this form it is quite possible
- One property cannot be updated more than once per second - it can be solved by adding 3 more characters for milliseconds.

## Usage

### Class example `Temporal.Simple`.

1. To start with, generate sample data: `do ##class(Temporal.Simple).populate(count, updCycles, rebuild)` Creates `count` records, updates each `updCycles` times, deletes all data if `rebuild=1`. By default, it creates 1kk records, updating each one 10 times.
2. To open an object at a certain moment of time (ro) run: `set obj = ##class(Temporal.Simple).openAt(id, concurrency, .sc, state, timestamp, debug)`, where:
- `id` - the primary key
- `concurrency` [-1] - object opening mode
- `sc` - status
- `state` [1] - what value to take if there is no value on timestamp. 1 - the next timestamp, -1 - the previous
- `timestamp` [now] - date/time in Timestamp format: 2017-01-19 00:18:14
- `debug` [0] - if 1, then timestamp in the unixtime format: 1484838070

3. To execute a query for a certain time, add `Temporal.AtTime(state, timestamp, debug)=1` condition to WHERE, the variables are similar to 2.
4. To open the latest version of the object (rw) use %OpenId(id, concurrency, .sc)

## TODO

- Unify %ts, ..%ts, timestamp
- Generators how?
   + In the class
   + In the special type of data from which the special data types are inherited?

# Temporal
Хранение данных с историей в Caché.

# Установка

1. Загрузите пакет Temporal и скомпилируйте его

# Идея

Каждое свойсво Property хранится в отдельном узле: `Package.ClassD.PropertyD(id, timestamp) = val` где:
  - Package.ClassD - глобал данных класса
  - Package.ClassD.PropertyD - глобал свойства Property
  - id - первичный ключ
  - timestamp - дата/время в формате [unixtime](https://en.wikipedia.org/wiki/Unix_time)
  - val - значение свойства (Для типа данных пишем значение, для объекта - id, сериализуемые сериализуем, стримы оборачиваем в классы и пишем id, списки/массивы объектов сериализуем в списки/массивы id, а дальше списки/массивы данных сериализуем)
 
Также автоматически генерируется узел: `Package.ClassD.PropertyD(id) = val` с актуальным значением свойства.

# Геттер свойства

## Объектный контекст

Загрузка в память идёт при первом обращении, дальше берём из памяти. Алгоритм такой:

1. Если значение есть в памяти, то берём его. ВЫХОД
2. Если нужна последняя версия данных, то берём её
3. Если нужны данные на момент %ts, то пытаемся взять данные именно на момент %ts
4. Если не нашли 3, то берём ближайшее следующее/предыущее значение
5. Если нужно значение на дату >%ts, но его не существует, то берём предыдущее значение (должно быть аналогично 2? TODO)
6. Записываем полученное значение в память
7. Возвращаем значение 

## SQL контекст

Для SQL всё аналогично, но выполняются только пункты  2-5,7.

  
# Ограничения
 
 - Нет INSERT/UPDATE - в принципе можно написать триггер
 - Не решена проблема одновременного обновления - Lock придумали как раз для этого
 - Нет индексов на текущие значения - не вижу как решать
 - Временные переменные %ts/%state при использовании в COS SQL надо либо new до либо удалять после
 - Нет хранения отметок изменения объектов - Надо? Use case? Как? Проецировать в класс `{class, id, ts, diff}`. В таком виде сделать представляется вполне возможным
 - Одно и то же свойство нельзя обновлять чаще чем 1 раз в секунду - решается добавлением ещё 3 знаков для миллисекунд - это есть в Caché и в коде закоментировано
 

# Использование

Класс-пример `Temporal.Simple`. 

1. Для начала сгенерируйте данные: `do ##class(Temporal.Simple).populate(count, updCycles, rebuild)` Cоздает count записей, обновляет значение каждой updCycles раз, удаляет все данные, если установлен rebuild=1. По умолчанию создаёт 1kk записей, обновляя каждую по 10 раз.
2. Для открытия объекта на определённый момент времени (ro) выполните: `set obj = ##class(Temporal.Simple).openAt(id, concurrency, .sc, state, timestamp, debug)`, где: 
  - id - первичный ключ
  - concurrency [-1] - режим открытия объекта
  - sc - статус
  - state [1] - какое значение брать, если на timestamp значения нет. 1 - следующее, -1 - предыдущее
  - timestamp [now] - дата/время в формате Timestamp `2017-01-19 00:18:14`
  - debug [0] - если 1, то timestamp в фррмате unixtime `1484838070`
  
3. Для выполнения запроса на определённый момент времени, добавьте во WHERE условие `Temporal.AtTime(state, timestamp, debug)=1`, переменные аналогичны 2.
4. Для открытия последней версии объекта (rw) используйте `%OpenId(id, concurrency, .sc)`

# TODO
  - Унифицировать %ts, ..%ts, timestamp
  - Генераторы как?
      - В классе
      - В особом типе данных, от которого наследуются спецтипы данных? 



# Temporal
Store temporal data in Caché
