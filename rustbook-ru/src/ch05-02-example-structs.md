## Пример использования структур

Для понимания, где мы могли бы использовать структуры, мы напишем программу, которая будет рассчитывать площадь прямоугольника. Мы начнём с создания переменных, а потом постепенно напишем код, который будет использовать структуры.

Давайте создадим новый проект программы при помощи Cargo и назовём его *rectangles*. Наша программа будет получать на вход длину и ширину прямоугольника в пикселях и затем рассчитывать площадь прямоугольника. Листинг 5-8 показывает один из коротких вариантов кода который позволит нам сделать именно то, что надо, код в файле проекта *src/main.rs*.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```

<span class="caption">Листинг 5-8: Расчёт площади прямоугольника с помощью отдельных переменных ширины и высоты</span>

Теперь, проверим её работу `cargo run`:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

Несмотря на то, что код листинга 5-8 работает и рассчитывает площадь прямоугольника вызывая функцию `area` для каждого изменения, мы можем улучшить программу. Переменные длины и ширины связаны логически, так как они совместно описывают параметры прямоугольника.

Проблема данного метода очевидна из сигнатуры `area`:

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

Функция `area` должна рассчитывать площадь одного прямоугольника, но у функции описано два параметра. Эти параметры связаны логически, но это никак не отражено в коде программы. Код был бы более очевидным и управляемым, если бы переменные ширины и длины были сгруппированы вместе. Мы уже знаем один из методов группировки переменных из раздела ["Тип кортеж"]<!--  --> Главы 3, в котором используются кортежи.

### Рефакторинг при помощи кортежей

Листинг 5-9 это другая версия программы, использующая кортежи.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```

<span class="caption">Листинг 5-9: Указание ширины и высоты прямоугольника с помощью кортежа</span>

С одной стороны, эта программа стала лучше. Кортежи позволяют лучше структурировать код, теперь мы передаём один аргумент. Но с другой стороны данная версия менее понятная: кортежи не имеют имён для элементов, так что расчёт стал более запутанным из-за необходимости индексирования частей кортежа.

Если мы перепутаем местами ширину с высотой при расчёте площади, то это не имеет значения. Но если нужно нарисовать прямоугольник на экране, то это уже будет иметь значение! Придётся помнить, что ширина  `width` находится в кортеже с индексом `0`, а высота `height` с индексом `1`. Если кто-то другой поработал бы с кодом, ему бы пришлось разобраться в этом и также помнить про порядок. Легко забыть и перепутать эти значения и это вызовет ошибки, потому что данный код не передаёт наши намерения.

### Рефакторинг при помощи структур: добавим больше смысла

Мы используем структуры чтобы добавить смысл данным при помощи назначения им осмысленных имён . Мы можем переделать используемый кортеж в структуру: тип данных (data type) с единым именем для сущности и частными названиями её частей, как показано в листинге 5-10.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

<span class="caption">Листинг 5-10: Определение структуры <code>Rectangle</code></span>

Здесь мы определили структуру и дали ей имя `Rectangle`. Внутри фигурных скобок определили поля как `width` и `height`, оба - типа `u32`. Затем в `main` создали конкретный экземпляр `Rectangle` с шириной в 30 и высотой в 50 единиц.

Наша функция `area` теперь определена с одним параметром названным `rectangle`, чей тип является неизменяемым заимствованием структуры `Rectangle`. Как упоминалось в Главе 4, необходимо заимствовать структуру, а не передавать её во владение. Таким образом функция `main` сохраняет `rect1` в собственности и может её использовать дальше, по этой причине мы и используем `&` в сигнатуре и в месте вызова функции.

Функция `area` имеет доступ к полям `width` и `height` экземпляра `Rectangle`. Сигнатура нашей функции для `area` теперь точно говорит, что мы имели ввиду: посчитать площадь `Rectangle` используя поля `width` и `height`. Такой подход сообщает, что ширина и высота связаны по смыслу друг с другом. А названия значений структуры теперь носят понятные описательные имена, вместо ранее используемых значений индексов кортежа `0` и `1`. Это плюс к ясности.

### Добавление полезной функциональности при помощи Выводимых Типажей

Было бы не плохо иметь возможность печатать экземпляр `Rectangle` во время отладки программы и видеть значения всех полей. Листинг 5-11 использует макрос `println!`, который мы уже использовали в предыдущих главах. Тем не менее, это не работает.

<span class="filename">Файл: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```

<span class="caption">Листинг 5-11: Попытка распечатать экземпляр <code>Rectangle</code></span>

При компиляции этого кода мы получаем ошибку с сообщением:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

Макрос `println!` умеет делать разные виды форматирования по умолчанию, фигурные скобки в `println!` говорят использовать форматирование известное как типаж `Display`: вывод в таком варианте форматирования предназначен для прямого и пользовательского потребления. Примитивные типы изученные ранее, по умолчанию реализуют типаж `Display`, потому что есть только один способ отобразить число `1` или любой другой примитивный тип пользователю. Но для структур у которых `println!` должен форматировать способ вывода данных, это является менее очевидным, потому что есть гораздо больше возможностей для отображения: Вы хотите запятые или нет? Вы хотите печатать фигурные скобки? Нужно ли показать все поля? Из-за этой неоднозначности Rust не пытается  угадать, что нам нужно -  структуры не имеют готовой реализации типажа `Display`.

Продолжив чтение текста ошибки, мы найдём полезное замечание:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

Давайте попробуем! Вызов макроса `println!` теперь будет выглядеть так `println!("rect1 is {:?}", rect1);`. Ввод спецификатора `:?` внутри фигурных скобок говорит макросу `println!`, что мы хотим использовать другой формат вывода известный как `Debug`. Типаж `Debug` позволяет печатать структуру способом, удобным для разработчиков, чтобы видеть значение во время отладки кода.

Скомпилируем код с этими изменениями. Упс! Мы всё ещё получаем ошибку:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

Снова компилятор даёт нам полезное замечание:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

Rust *&nbsp;реализует* функциональность для печати отладочной информации, но *не включает (не выводит) её по умолчанию* , мы должны явно включить эту функциональность для нашей структуры. Чтобы это сделать, добавляем аннотацию `#[derive(Debug)]` сразу перед определением структуры как показано в листинге 5-12.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/src/main.rs}}
```

<span class="caption">Листинг 5-12: Добавление аннотации для вывода типажа  <code>Debug</code> и печати экземпляра <code>Rectangle</code> с отладочным форматированием</span>

Теперь при запуске программы мы не получим ошибок и увидим следующий вывод:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

Отлично! Это не делает вывод приятнее, но показывает значения всех полей экземпляра, которые определённо помогут при отладке. Когда у нас структуры больше, то полезно иметь более простой для чтения вывод; в таком случае можно использовать код `{:#?}` вместо ` {:?}` в строке макроса `println!`. При использовании стиля `{:#?}` в примере вывод будет выглядеть так:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

Rust предоставляет много типажей для использования в аннотации `derive`, они умеют давать полезное поведение пользовательским типам. Эти типажи и их поведение перечислены в приложении C. Мы обсудим как реализовать поддержку типажей для нашего кода с индивидуальным поведением, а также как создавать свои собственные типажи в Главе 10.

Функция `area` является довольно специфичной: она считает только площадь прямоугольников. Было бы полезно привязать данное поведение как можно ближе к структуре `Rectangle`, потому что наш специфичный код не будет работать с любым другим типом. Давайте рассмотрим, как можно улучшить наш кода превращая функцию `area` в <em>метод</em> <code>area</code>, определённый для типа `Rectangle`.


["Тип кортеж"]: ch03-02-data-types.html#the-tuple-type