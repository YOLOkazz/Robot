# ПОДКЛЮЧЕНИЕ И КОМАНДНЫЙ ИНТЕРФЕЙС РОБОТА

## Подключение Робота

Для начала работы с Роботом требуется только выполнить включение содержимого файла `robot.jl` в область REPL:

    julia> include("robot.jl")

(выполнять эту команду необходимо в начале каждого нового сеанса работы с Julia).

После этого включения имена `HorizonSide`, `Nord`, `West`, `Sud`, `Ost`, `Robot`, `move!`, `isborder`, `putmarker!`, `ismarker`, `temperature`, `show`, `save`, а также `sitedit` и `sitcreate` окажутся доступными в соответствующем пространстве имен.  В результате, в частности, станет возможным получить более детальную информацию о каждом из них с помощью встроенной системы помощи (help).

Стоит иметь в виду, что, поскольку функция `include` (как бы) осуществляет вставку соответствующего программного кода из указанного файла в текущую позицию, то после выполнения `include("robot.jl")` более одного раза во время сеанса могут возникнуть проблемы, связанные с дублированием некоторых определений.

## Получение справки

Чтобы из стандартного режима REPL перейти в режим справки, потребуется ввести команду:

    julia> ?

После этого можно будет, например, набрать:

    help?> Robot

и получить интересующую информацию. Вернуться в обычный режим можно нажатием кнопки `Backspace`.


Файл `robot.jl` содержит определение модуля `HorizonSideRobot`, включающего в себя определения следующих экспортируемых типов данных

* HorizonSide

    @enum HorizonSide Nord=0 West=1 Sud=2 Ost=3

    Это перечисление, определяющее обозначения сторон горизонта (Cевер, Запад, Юг, Восток, соответственно). Фактически, `HorizonSide` - это тип данных с возможными значениями: `Nord`, `West`, `Sud`, `Ost`, при этом `HorizonSide(0)` означает `Nord`, `HorizonSide(1)` — `West` и так далее.

* Robot
  
    Данный пользовательский тип предназначен для создания (с помощью конструктора `Robot`) объектов — «Роботов на клетчатом поле со сторонами горизонта». Каждый такой объект позволяет имитировать программное управление роботом, движущимся по клетчатому полю с заданной обстановкой и выполняющим на нем те или иные действия.

В дальнейшем, уже для работы с Роботом, импортировать модуль HorizonSideRobot не придется, так как соответствующий оператор импортирования уже содержится в конце файла robot.jl. Достаточно будет простого включения содержимого файла с помощью `include`, см. ниже.

## Создание объекта типа Robot

Объект типа Robot предназначен для имитации управления роботом, способным перемещаться по клетчатому полю и выполнять еще некоторые действия.

Обстановка на поле определяется наличием или отсутствием ограничивающей поле прямоугольной рамки (поле может быть ограниченным или неограниченным), наличием или отсутствием внутренних (межклеточных) перегородок, наличием или отсутствием маркеров в клетках поля, положением робота на поле. Робот не может убирать имеющиеся перегородки или устанавливать новые, но может проверять их наличие, он может делать шаг ровно на 1 клетку в кажом из 4-х направлений горизонта (наличие на пути перегородки приведет к фатальной ошибке), может устанавливать маркер в клетке поля, в которой робот находится в данный момент, проверять наличие маркера в этой клетке, но убрать ранее поставленный маркер он не может. Также робот умеет измерять температуру в той клетке поля, в которой находится.

Предусмотрены следующие варианты вызова конструктора Robot.

1. `r=Robot()`
2. `r=Robot(<число_строк_клеток_поля>,<число_столбцов_клеток_поля>)`
3. `r=Robot(<имя_файла_с_oбстановкой>)`
4. `r=Robot(...; animate=true)`

В первом случае создаётся ограниченное рамкой поле 11x12 без внутренних перегородок и маркеров в клетках, с положением робота в юго-восточном (нижнем левом) углу.

Во втором — поле будет иметь указанные размеры.

В третьем — обстановка на поле будет соответствовать данным, загруженным из соответствующего файла; рекомендуется для таких файлов использовать расширение `.sit` (от слова situation).

Во всех трёх случаях создается объект `r` типа `Robot` (точнее говоря, `r` — это ссылка на созданный объект), внутри себя содержащий структуру данных с соответствующей обстановкой. При выполнении команд робота эта внутренняя структура данных может изменяться, но визуализации обстановки на поле при этом ещё не будет.

Для просмотра текущей обстановки следует воспользоваться функцией `show(r)`. В результате её выполнения будет открыто графическое окно с текущей обстановкой. При необходимости во время просмотра редактировать текущую обстановку с помощью мыши, вместо функции `show` следует использовать функциию `show!`.

Сохранить текущую обстановку в файле можно вызовом функции `save(r, <имя_файла>)`.

Кроме этого, имеется возможность запустить робота с анимацией выполняемых им действий, указав значение `true` для параметра `animate`:  

    julia> r=Robot(...; animate=true)

где вместо многоточия должны быть указаны фактические параметры (или они могут отсутствовать) в соответствии с первыми 3-мя вариантами использования конструктора. 

При этом функции `show` и `show!` использовать будет уже нельзя (приведёт к ошибке времени выполнения). Но все функции графического окна, открывавшегося прежде при вызове функции `show` или `show!`, будут присущи аналогичному графическому окну, открывающемуся теперь автоматически при вызове конструктора.

С помощью этого окна можно отслеживать все действия Робота в режиме анимации. Однако этот режим приводит к дополнительным временным задержкам, не желательным, может быть, при выполненнии Роботом объёмных действий.

## Командный интерфейс Робота

Перечень функций, составляющих командный интерфейс Робота — с помощью которого, и только с помощью которого, с Роботом можно взаимодействовать:

- `Robot(...)` — создаёт Робота.
- `move!(::Robot,::HorizonSide)::Nothing` — перемещает робота ровно на 1 клетку в указанном направлении.
- `isborder(::Robot,::HorizonSide)::Bool` — проверяет наличие перегородки в указанном направлении.
- `putmarker!(::Robot)::Nothing` — ставит маркер в клетке с роботом.
- `ismarker(::Robot)::Bool` — проверяет наличие маркера в клетке с роботом.
- `temperature(::Robot)::Int` — возвращает температуру клетки с роботом.
- `show(::Robor)::Nothing` — открывает окно с обстановкой на поле (без возможности редактирования).
- `show!(::Robor)::Nothing` — открывает окно с обстановкой на поле и предоставляет возможность редактирования обстановки.
- `save(::Robot, ::AbstractString)` — сохраняет текущую обстановку в указанном файле.

(здесь всюду на позициях параметров каждой из функций указаны только типы параметров)

По любой из функций можно получить исчерпывающую информацию, используя встроенную систему помощи (но только после того как будет выполено `include("robot.jl")`), например:

    julia> ?
    help?> move!
        move!(r::Robot, side::HorizonSide)::Nothing
    -- Перемещает робота в соседнюю клетку в заданном направлении (если только на пути нет перегoродки, в противном случае - прерывание)

После созданния объекта типа `Robot` (и записи ссылки на него в некоторую переменную, обычно называемую `r`) любую команду Робота можно исполнить непосредственно из REPL. Для наибольшей наглядности можно воспользоваться возможнотью запуска Робота в интеракивном режиме. Например:

    julia> r=Robot(animate=true);
    
    julia> move!(r,Nord)
    
    julia> isborder(r,Ost)
        true
    julia> ismarker(r)
        false
    julia> putmarker(r)
    
    julia> ismarker(r)
        true
    julia> temperature(r)
        42

Точка с запятой после вызова конструктра `Robot` поставлена, чтобы предотвратить вывод на экран не нужной нам информации о внутренней структуре созданного объекта.

--------------------------------------------------------

Начальные сведения о языке Julia, вполне достаточные для понимания разбираемого далее примера программы, и для начала выполнения заданий для самостятельной работы содержатся [здесь](language.md)
