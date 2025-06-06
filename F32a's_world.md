https://github.com/ryukzak/wrench/blob/master/src/Isa/F32a.hs

Готовьтесь к слегка токсичному повествованию, это позволит вам передать вайбы этой архитектуры

Стековая архитектура, богиня дискотеки, мисс синтаксис года

Здесь есть два стека: стек данных и стек возврата. Первый используется для хранения данных, второй для хранения адреса возврата, используется в немногих случаях, к примеру, для реализации call и next. Можно делать push для загрузки на стек и pop для чтения элемента со стека

Используемые регистры:
Регистр p (не путать с p = pc то есть счетчиком команд), a и b
Регистр a используется для хранения адресов и данных, регистр b только для хранения адресов, а регистр p позволяет обращаться к памяти и писать в нее без сохранения в регистр (подобнее в описании команд). Логично, что у нас больше регистров для хранения адресов, так как для данных есть целый стек.

Если в регистре хранится адрес, то можно читать данные из него, то есть делать fetch (@), или писать в него, то есть делать store(!)

Всегда нужно понимать, что и в какой последовательности лежит в стеке и как команды изменяют его содержимое

К синтаксису нужно привыкнуть, будьте готовы к тому, что несколько команд находятся в одной строке (это удобно, на самом деле, если воспринимать команды в строке в совокупности). Есть варик посмотреть, как Пенской смешно ахуевает с программы, которая фул написана в одну строчку, но это опасно для вашего хп, выполнять строго под присмотром взрослых!

Для этой архитектуры есть обязательное задание - реализовать вызов процедуры. Теоретические сведения об этом в гайде с ответами на вопросы, на практике процедуры релизуются набором call, метка и return. Команда call по синтаксису выглядит, как стоящее в соло название метки в строке, return по синтаксису это тупа ```;```. 

Часто встречается случай, когда нужно проделать какие-то проверки с верхним элементом стека данных и при этом дальше нужно каким-то образом взаимодействовать с этим значением, поэтому используется команда dup, которая дублирует верхний элемент и все мувы производятся с ним, в конце обычно делается проверка if'ом, который делает pop из стека данных, и в итоге остается лишь сохраненное ранее значение

Количество условных переходов тут всего 2 (> 0 и >= 0), а количество раз, сколько у меня из-за этого сгорало, стремится к бесконечности)


Проверку на окончание цикла for нужно делать в конце итерации. Есть четыре варианта. Первый, завести отдельную переменную под n + 1 и после инкремента счетчика цикла с помощью xor сравнивать его с этой переменной, если результат 0, то счетчик цикла достиг конца и выходим из него. Второй вариант не заводить эту переменную, а просто делать xor с n перед инкрементом счетчика цикла, n мы уже обработали, и это последнее значение, которое надо обработать, поэтому выходим из цикла. Третий вариант загрузить в стек сразу n и уменьшать его каждую итерацию на 1, в конце итерации если стало 0 это число, то выходим. Четвертый вариант для самых сильных, не выходить из цикла, ведь бесконечность не предел!

EAM - extended arithmetic mode -  он позволяет нам совершать операцию сложения add с добавлением к результату carry флага, если предыдущий результат сложения превысил размер 32 бита, то есть флаг carry был установлен. Чтобы понять, как это работает посмотрите программу и результаты её тестирования

https://github.com/ryukzak/wrench/blob/master/test/golden/f32a/carry.s

https://github.com/ryukzak/wrench/blob/master/test/golden/f32a/carry.f32a.result

Тут видно, что после сложения 0xffffffff и 0x1 получилось 0x0 и флаг carry установился в 1, следующая операция сложения 0x0 и 0x2 (его загрузили заранее) дала результат 0x3 и carry обнулился (такая же прога, но без включенного eam дала в конце результат 0x2). Да, всрато, но как есть.
![image](https://github.com/user-attachments/assets/7436abc3-3bd1-47a3-813c-be79603acdff)


Операции деления и умножения здесь тоже всратые, так как есть команды только для проведения лишь 1 шага умножения и деления. Разбор этих операций в другом гайде.


## Описание команд (обязательно обращайте внимание на операции push и pop в секции что делает, чтобы понимать, как меняется содержимое стека): 

# ```lit```
Что делает:
```
dataStack.push(<value>)
```

Пояснение:
Пушится в стек данных число напрямую

Пример использования:
```
lit 10
```

Что произошло после выполнения:
На вершине стека 10 в десятичной системе. Может также быть ```lit random_label``` тогда загружается адрес метки на стек, ПАСИБА

# ```left shift```
Что делает:
```
dataStack.push(dataStack.pop() << 1)
```
Пояснение:
Сдвиг верхнего значения стека на 1 бит влево, то есть по-простому умножение на 2
Пример использования:
```
lit 10
2*
```
Что произошло после выполнения:
В стек загрузили 10, в двоичной это 1010, сдвигаем на 1 влево  - 10100, это число 20 в десятичной

# ```right shift```
Что делает:
```
dataStack.push(dataStack.pop() >> 1)
```
Пояснение:
Сдвиг верхнего значения стека на 1 бит вправо, то есть по-простому деление на 2
Пример использования:
```
lit 10
2/
```
Что произошло после выполнения:
В стек загрузили 10, в двоичной это 1010, сдвигаем на 1 вправо  - 101, это число 5 в десятичной
# ```invert```
Что делает:
```
dataStack.push(~dataStack.pop())
```
Пояснение:
Инвертирует все биты числа с верхушки стека
Пример использования:
```
lit -1
inv
```
Что произошло после выполнения:
-1 как мы знаем это 32 единицы, инвертируем их и получаем 0 на верхушке стека
# ```eam```
Что делает:
```
eam <- dataStack.pop()
```
Пояснение:
Кладем на стек 0, если хотим выключить EAM mode, и 1, если хотим включить EAM mode. 
Пример использования:
```
\ о, это коммент, кстати
lit 1 eam
```
Что произошло после выполнения:
Со стека сняли 1, положили его в eam, тем самым, включив EAM mode.

# ```call```
Что делает:
```
returnStack.push(p); p <- <label>
```

Пояснение:
Используется для вызова выполнения удаленной программы, при этом значения счетчика команд пушится в стек возврата, синтаксис просто название метки в строке без всяких других символов

Пример использования:
```
    do_work
    @ - @
    halt
do_work:
    lit 0
    if callback
callback:
    ;
     
```

Что произошло после выполнения:
Мы прыгнули на do_work, при этом в стек возврата записался адрес команды call (не путать со стеком данных)


# ```return```
Что делает:
```
p <- returnStack.pop()
```

Пояснение:
Используется для возврата к следующей команде после call, синтаксис просто точка с запятой ```;```
Метка callback используется, чтобы выходить по условию, ведь нельзя написать ```if ;``` так как это воспримется интерпретатором как команда jump, поэтому заводим отдельную метку, куда прыгнем по условию и там прожмем return

Пример использования:
```
    do_work
    @ - @
    halt
do_work:
    lit 0
    if callback
callback:
    ;
```

Что произошло после выполнения:
Помним, что в стеке возврата лежал адрес call, return его берет и переходит к следующей после call команде


# ```jump```
Что делает:
```
p <- <label>
```

Пояснение:
Безусловный переходит, по синтаксису название метки,  пробел, затем точка с запятой

Пример использования:
```
    kiss_my_ass ;
kiss_my_ass:
    \чмок
    halt
```

Что произошло после выполнения:
Сделали переход на метку

# ```next```
Что делает:
```
if R != 0 then R <- R - 1; p <- <label>
```

Пояснение:
Кароче эта вкусняшка нужна, чтобы не плодить условия и, к примеру, хранить в стеке возврата количество итераций цикла, будет происходить переход на метку до тех пор, пока счетчик цикла не станет нулем, после этого будет произведен переход к следующей после next команде. Используется в случае, если нужно совершить столько-то итераций цикла, но при этом нам не требуется обрабатывать значение счетчика цикла, в наших же вариках чаще всего это делать нужно. Если нужно сделать 32 итерации, то в стек возврата нужно записывать число на 1 меньше, то есть 31 в данном случае. В стековой архитектуре также полезна для совершения сдвигов на несколько битов влево или вправо.

Пример использования:
```
    lit 31 >r                     \ push 31 to return stack

step:
    lit 1 +                        \ к верхнему элементу стека данных будет прибавлено 1

    next step
    halt
```

Что произошло после выполнения:
32 раза к верхнему элементу стека будет прибавлена единица, после того как R станет 0 мы перейдем к halt. Кстати, команда ```>r``` берет значение со стека данных и пушит его в стек возврата, а ```r>``` берет значение стека возврата и пушит его в стек данных

# ```if```
Что делает:
```
if dataStack.pop() == 0 then p <- <label>
```

Пояснение:
Берется верхний элемент стека данных если он 0, то переход на метку (стек стал на 1 элемент меньше)

Пример использования:
```
lit 0
if equals_zero
equals_zero:
       halt
```

Что произошло после выполнения:
Будет совершен переход, так как на вершину стека мы положили 0

# ```-if```
Что делает:
```
if dataStack.pop() >= 0 then p <- <label>
```

Пояснение:
Переход, если на вершине стека число большее или равное 0, спасибо папаша оч удобно

Пример использования:
```
lit 1
-if equals_or_greater_zero
equals_or_greater_zero:
       halt
```

Что произошло после выполнения:
Будет произведен переход

# ```AStore```
Что делает:
```
A <- dataStack.pop()
```

Пояснение:
Берется значение со стека данных и кладется в регистр a

Пример использования:
```
lit some_label a!
```

Что произошло после выполнения:
Мы загрузили на стек данных адрес метки и потом положили его в регистр а (я говорил про запись в одну строку, это как раз пример почему так удобно писать)

# ```BStore```
Что делает:
```
B <- dataStack.pop()
```

Пояснение:
То же самое, что и с AStore

Пример использования:
```
lit 10 b!
```

Что произошло после выполнения:
Ну, тут по приколу загрузили число в 10 в регистр b, ну а че, можем себе позволить

# ```AFetch```
Что делает:
```
dataStack.push(A)
```

Пояснение:
Здесь начинаются различия с регистром b, ведь у него нет такой команды. Берется значение из а и пушится на стек. Это пример, когда регистр а используется для хранения данных, а не адреса

Пример использования:
```
lit 69 a!
a
```

Что произошло после выполнения:
На верхушку стека запушится 69, которое мы загрузили в регистр a

# ```Fetch```
Что делает:
```
dataStack.push(mem[A])
```

Пояснение:
В регистре а лежит адрес и мы получаем слово, которое находится по этому адресу

Пример использования:
```
.data
input_addr:    0x80

.text
_start:
lit input_addr a!
@
```

Что произошло после выполнения:
На стек запушится 0x80 (само значение адреса, а не ввод), так как это содержимое адреса, которое перед было загружено в регистр a (для получения input будет более эффективный способ с помощью FetchP) 


# ```FetchPlus```
Что делает:
```
dataStack.push(mem[A]); A <- A + 1

```

Пояснение:
В регистре a лежит указатель, как и в Fetch, после чтения из памяти по этому адресу значение этого указателя увеличивается на 1, полезно при работе со строками, к примеру, когда нужно считать символ и потом перейти к обработке следующего 

Пример использования:
```
.data
hi:            .byte 'hi'
input_addr:    .word 0x80
output_addr:   .word 0x84



.text
_start:
      @p output_addr b!
      lit hi a!
      @+
      lit 0xFF and
      !b
      @+
      lit 0xFF and
      !b
      halt
```

Что произошло после выполнения:
Это полный код программы для вывода на вывод строки символов 'hi' в виде ascii кодов в 0x84. Чекните FetchP и возвращайтесь сюда. С возвращением! В регистр b было загружено 0x84, а в регистр a загружен указатель на начало строки hi. @+ считывает целое слово вместо байта, поэтому надо делать маску на байт умножением на 0xFF, после этого указатель в a увеличивается на 1 теперь указывает на второй символ. Чекните StoreB и возвращайтесь сюда. С возвращением!! Пиишем в 0x84 полученный символ.

# ```FetchB```
Что делает:
```
dataStack.push(mem[B])
```

Пояснение:
То же самое, что и Fetch

Пример использования:
```
.data
input_addr:    0x80

.text
_start:
lit input_addr b!
@
```

Что произошло после выполнения:
Аналогично FetchA. Удобно сочетать, к примеру, в регистре a хранить указатель, куда писать, а в регистре b хранить, откуда читать

# ```FetchP```
Что делает:
```
dataStack.push(mem[<address>])
```

Пояснение:
Отличается от Fetch и FetchB тем, что не происходит чтения регистра, а напрямую читается содержимое метки и пушится на стек

Пример использования:
```
.data
input_addr:    0x80

.text
_start:
@p input_addr a!
```

Что произошло после выполнения:
Мы прочитали содержимое input_addr и загрузили его в a, то есть в а лежит 0x80. Если далее делать Fetch, то мы как раз получим слово из ввода

# ```StoreP```
Что делает:
```
mem[<address>] <- dataStack.pop()
```

Пояснение:
Со стека берется элемент и пишется в содержимое метки, также без обращения к регистрам

Пример использования:
```
@p counter
lit 1 +
!p counter
```

Что произошло после выполнения:
Мы сначала получили содержимое counter, потом увеличили его на 1 и после сохранили его в counter. Удобно использовать, когда не нужно долго хранить какой-то адрес, секс на одну ночь кароч

# ```StorePlus```
Что делает:
```
mem[A] <- dataStack.pop(); A <- A + 1

```

Пояснение:
По примеру FetchPlus думаю разберетесь, также со строками, к примеру, чтобы заполнять посимвольно выделенный буфер в памяти 


Пример использования:
```
.data
hi:            .byte '--'
input_addr:    .word 0x80
output_addr:   .word 0x84



.text
_start:
      @p input_addr b!
      lit hi a!
      @b
      lit 0xFF and
      !+
      @b
      lit 0xFF and
      !+
      halt
```

Что произошло после выполнения:
С 0x80 последовательно считываем символ 'h' и потом 'i', в регистре a лежит указатель на начало hi, который еще пока не заполнен символами, но потом туда будут поочередно писаться символы

# ```StoreB```
Что делает:
```
mem[B] <- dataStack.pop()
```

Пояснение:
Берется элемент с верхушки стека данных и пишется в адрес, который лежит в регистре b


Пример использования:
```
@p output_addr b!
lit 666
!b
```

Что произошло после выполнения:
В 0x84 будет выгружена татушка Моргена (блин, шутка потеряла свою актуальность в этом году)

# ```Store```
Что делает:
```
mem[A] <- dataStack.pop()

```

Пояснение:
По примеру StoreB

Пример использования:
```
@p output_addr a!
lit siski
!
```

Что произошло после выполнения:
siski это метка, помним, что lit label загружает на стек значение ее адреса, в 0x84 это и пойдет

# ```add```
Что делает:
```
\ no EAM mode
dataStack.push(dataStack.pop() + dataStack.pop()), set carry flag
\ EAM mode
dataStack.push(dataStack.pop() + dataStack.pop() + carry), set carry flag
```

Пояснение:
Видим, что на стеке должны быть оба операнда, они снимаются с вершины, складываются и на стек идет только результат (кол-во элементов стека уменьшилось на 1). Соответственно, если нужно сохранить какой-то операнд делаем dup, потом загружаем сверху второй операнд и в конце либо убираем результат с помощью условия, либо с помощью drop
Поддерживается EAM mode. К примеру, мы складываем два числа, сумма которых превышает 32 бита, будет установлен Carry флаг, вместе со следующей операцией сложения этот carry флаг будет добавлен к результату сложения. К примеру, 1 + 1 будет 3, вот наглядный пример add с carry флагом:
![image](https://github.com/user-attachments/assets/764e2595-c93c-4a66-bc5b-413f5fe99836)


Пример использования (no EAM):
```
lit 0 eam
lit 0
lit 2
lit 0xFFFFFFFF +
+ 

```

Что произошло после выполнения:
после команды lit 0xFFFFFFFF + 
carry: 1 eam: 0 stack: 0x00000001:0x00000000 (0x00000000 - это тот нолик, что мы загрузили перед lit 2)
после команды +
carry: 0 eam: 0 stack: 0x00000001


Пример использования (EAM):
```
lit 1 eam
lit 0
lit 2
lit 0xFFFFFFFF +
+ 

```

Что произошло после выполнения:
после команды lit 0xFFFFFFFF + 
carry: 1 eam: 1 stack: 0x00000001:0x00000000 (0x00000000 - это тот нолик, что мы загрузили перед lit 2)
после команды + (последняя строчка)
carry: 0 eam: 0 stack: 0x00000002


# ```drop```
Что делает:
```
dataStack.pop()
```

Пояснение:
Тупа выкидывает вершину стека данных 

Пример использования:
```
lit 10
lit 15
drop
```

Что произошло после выполнения:
На стеке останется лишь 10

# ```dup(степ ахах простите)```
Что делает:
```
dataStack.push(dataStack.top()), carry flag does not change
```

Пояснение:
Дублирует вершину стека

Пример использования:
```
lit 1
dup
```

Что произошло после выполнения:
На стеке теперь две единицы, ну про мотивацию использовать dup вы уже начитались выше
Еще эта команда бережно относится с нашим carry флагом, оставляя его неизменным

# ```over```
Что делает:
```
T <- dataStack.pop(); S <- dataStack.pop(); dataStack.push(T); dataStack.push(S)
```

Пояснение:
Меняет местами два верхних элемента стека данных 

Пример использования:
```
lit 10
lit 15
over
```

Что произошло после выполнения:
Теперь верхушка стека это 10, а после него стоит 15 (помните, то, что мы загрузили на стек позже, в стеке будет находиться выше)


# ```xor```
Что делает:
```
dataStack.push(dataStack.pop() ^ dataStack.pop())
```
Пояснение:
Побитовое исключающее или, полезно, к примеру, в случае, если нам нужно проверить элемент на равенство чему-либо, если они будут равны, то их xor даст 0 в результате
Пример использования:
```
loop:
  lit 10
  lit 10
  xor
  if end
  loop;

end:
  halt
```

Что произошло после выполнения:
Мы не попадем в бесконечный цикл, так как 10 xor 10 дадут 0, if переведет нас на метку end


# ```and```
Что делает:
```
dataStack.push(dataStack.pop() & dataStack.pop())
```
Пояснение:
Побитовое умножение, полезно, к примеру, в случае, если нам нужно проверить четность числа, делаем and числа и 1, получая таким образом последний бит числа, если он 1, то число нечетное, иначе четное
Пример использования:
```
loop:
  lit 10
  lit 1
  and
  if even_number
  loop;

even_number:
  halt
```

Что произошло после выполнения:
Мы не попадем в бесконечный цикл, так как 10 and 1 дадут 0, if переведет нас на метку с названием, говорящим, что число четное.



