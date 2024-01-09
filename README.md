# CMake_like_a_guide
`При подготовке к экзамену по программированию собрал мини гайд по CMake, он не полный, можно дополнить связавшись со мной или сделав форк, буду рад если, кто-то поможет с этим`

# CMake как скриптовый язык  общего назначения
Скриптовый язык  общего назначения – язык програмирования выполняющий последовательные инструкции (сценарий), чаще всего интерпретируемый, нет чёткого назначения/универсальный

CMake имеет относительно простой интерпретируемый императивный язык сценариев, поддерживающий переменные, методы обработки строк, массивы, объявления функций и макросов, включение модулей (импортирование). Команды языка CMake (или директивы) считываются CMake из файла CMakeLists.txt. В этом файле указываются исходные файлы и параметры сборки, которые CMake размещает в спецификации сборки проекта (например, в make-файле). Кроме того, файлы с приставкой .cmake могут содержать сценарии, используемые CMake.

## Переменные
CMake работает в основном с переменными. Они задаются явно, записываются в результате выполнения команд, или используются как аргументы для команд. Переменная – это просто какое-нибудь имя и соответствующее ему значение. Внутри каждая переменная представляет собой просто строковое значение, и интерпретируется в зависимости от команды. Чтобы подставить значение переменной, используется конструкция ${VARNAME}, а для обращения к переменной (изменения) используется просто её имя. Переменные объявлять не надо, они создаются сами в момент первого обращения.
Надо понимать, что подстановка значения переменной через конструкцию ${} работает как простая текстовая замена.

Задать переменную явно:
```
set(VAR1 “text value”)
set(VAR2 VAR1)
set(VAR3 ${VAR1})
```
Переменные интерпретируются как логические значения, текстовые или списки. Истиной считается любое значение типа «1», «on», «y», «yes», «true» или ненулевое число. Все остальные значения считаются ложью.

### set
Список можно задать обычным set:
```
set(VARS file1 file2 file3)
```
Для работы со списками есть команда list, например:
```
list(LENGTH VARS VARS_LEN)
```
В переменную VARS_LEN будет записана длина списка VARS. Или
```
list(APPEND VARS "file4")
```
В список VARS будет добавлена строка «file4». Полный список действий можно посмотреть в официальной документации.

## Сообщения
Для вывода информации в консоль есть очень удобная команда message():
```
message(<mode> "message")
```
Тип сообщения может быть одним из следующих:
- `STATUS` – просто сообщение
- `WARNING` – предупреждение
- `AUTHOR_WARNING` – предупреждение разработчика
- `SEND_ERROR` – ошибка, при которой продолжается чтение файла, но прекращается генерация Makefile
- `FATAL_ERROR` – критическая ошибка, полная остановка
- `DEPRECATION` – предупреждение о чем-то устаревшем  
Напомню, что выводить содержимое переменной следует через ${}

## Условия
В CMake возможны условные ветки чтения CMakeLists.txt. для этого используется конструкция if-else-endif:
```
if (EXPRESSION)
  commands…
elseif (EXPRESSION2)
  commands…
else ()
  commands…
endif ()
```
В аргумент команды if() подается условие выполнения ветки. Это может быть переменная или логическое выражение, содержащее NOT, AND и OR. Для проверки существования переменной используется DEFINED.  
Примеры:
```
if (DEFINED DEBUG)
  message(STATUS "Building debug")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} –g3")
else ()
  message(STATUS "Building release")
endif ()
```
```
if(5 GREATER 1)
    message("Of course, 5 > 1!")
elseif(5 LESS 1)
    message("Oh no, 5 < 1!")
else()
    message("Oh my god, 5 == 1!")
endif()
```
```
# Напечатает в консоль три раза "VARIABLE is still 'Airport'":
set(VARIABLE Airport)
while(${VARIABLE} STREQUAL Airport)
    message("VARIABLE is still '${VARIABLE}'")
    message("VARIABLE is still '${VARIABLE}'")
    message("VARIABLE is still '${VARIABLE}'")
    set(VARIABLE "Police station")
endwhile()
```


## Функции в CMake
CMake позволяет объявлять функции командами function(name) / endfunction() и макросы командами macro(name) / endmacro(). Предпочитайте функции, а не макросы, т.к. у функций есть своя область видимости переменных, а у макросов - нет.
```
function(hello_get_something var_name)
  ...
  # Установить переменную в области видимости вызывающей стороны
  #  можно с помощью тега PARENT_SCOPE
  set(${var_name} ${ret} PARENT_SCOPE)
endfunction()
```
```
# Определить макрос, содержащий команду выхода:
macro(demonstrate_macro)
    return()
endmacro()

# Определить функцию, вызывающую предыдущий макрос:
function(demonstrate_func)
    demonstrate_macro()
    message("The function was invoked!")
endfunction()

# Напечатает "Something happened with the function!"
demonstrate_func()
message("Something happened with the function!")
```

## Работа с файлами
Также в CMake есть множество возможностей работы с файлами  
Все возможные вариации использования команды `file`:
```
Reading
  file(READ <filename> <out-var> [...])
  file(STRINGS <filename> <out-var> [...])
  file(<HASH> <filename> <out-var>)
  file(TIMESTAMP <filename> <out-var> [...])
  file(GET_RUNTIME_DEPENDENCIES [...])

Writing
  file({WRITE | APPEND} <filename> <content>...)
  file({TOUCH | TOUCH_NOCREATE} [<file>...])
  file(GENERATE OUTPUT <output-file> [...])
  file(CONFIGURE OUTPUT <output-file> CONTENT <content> [...])

Filesystem
  file({GLOB | GLOB_RECURSE} <out-var> [...] [<globbing-expr>...])
  file(MAKE_DIRECTORY [<dir>...])
  file({REMOVE | REMOVE_RECURSE } [<files>...])
  file(RENAME <oldname> <newname> [...])
  file(COPY_FILE <oldname> <newname> [...])
  file({COPY | INSTALL} <file>... DESTINATION <dir> [...])
  file(SIZE <filename> <out-var>)
  file(READ_SYMLINK <linkname> <out-var>)
  file(CREATE_LINK <original> <linkname> [...])
  file(CHMOD <files>... <directories>... PERMISSIONS <permissions>... [...])
  file(CHMOD_RECURSE <files>... <directories>... PERMISSIONS <permissions>... [...])

Path Conversion
  file(REAL_PATH <path> <out-var> [BASE_DIRECTORY <dir>] [EXPAND_TILDE])
  file(RELATIVE_PATH <out-var> <directory> <file>)
  file({TO_CMAKE_PATH | TO_NATIVE_PATH} <path> <out-var>)

Transfer
  file(DOWNLOAD <url> [<file>] [...])
  file(UPLOAD <file> <url> [...])

Locking
  file(LOCK <path> [...])

Archiving
  file(ARCHIVE_CREATE OUTPUT <archive> PATHS <paths>... [...])
  file(ARCHIVE_EXTRACT INPUT <archive> [...])
```
### Чтение из файла
```
file(READ <filename> <variable>
     [OFFSET <offset>] [LIMIT <max-in>] [HEX])
```
Считайте содержимое файла с именем <filename> и сохраните его в <variable> . При желании начните с заданного <offset> и прочитайте не более байтов <max-in> . Опция HEX приводит к преобразованию данных в шестнадцатеричное представление (полезно для двоичных данных). Если указана опция HEX , буквы в выводе (от a до f ) будут в нижнем регистре.
```
file(STRINGS <filename> <variable> [<options>...])
```
Разберите список строк ASCII из <filename> и сохраните его в <variable> . Двоичные данные в файле игнорируются. Символы возврата каретки ( \r , CR) игнорируются. Некоторые варианты опций:

- `LENGTH_MAXIMUM <max-len>` Рассматривайте только строки не более заданной длины.
- `LENGTH_MINIMUM <min-len>` Рассматривайте только строки хотя бы заданной длины.
- `REGEX <regex>` Рассматривайте только строки, соответствующие данному регулярному выражению, как описано в разделе string(REGEX) .
- `ENCODING <encoding-type>` Выбор кодировки документа

#### Примеры кода
`file(STRINGS myfile.txt myfile)`  
Хранит в переменной myfile список, в котором каждый элемент представляет собой строку из входного файла.

`file(<HASH> <filename> <variable>)`  
Вычислите криптографический хэш содержимого <filename> и сохраните его в <variable> . Поддерживаемые имена алгоритмов <HASH> перечислены в команде string(<HASH>).

`file(TIMESTAMP <filename> <variable> [<format>] [UTC])`  
Вычислите строковое представление времени модификации <filename> и сохраните его в <variable> . Если команде не удастся получить временную метку, ей будет присвоена пустая строка ("").
См. команду string(TIMESTAMP) для получения документации по опциям <format> и UTC .

## Работа со строками
В CMake есть большой функционал по работе со строками, все вариации работы с командой string
```
Search and Replace
  string(FIND <string> <substring> <out-var> [...])
  string(REPLACE <match-string> <replace-string> <out-var> <input>...)
  string(REGEX MATCH <match-regex> <out-var> <input>...)
  string(REGEX MATCHALL <match-regex> <out-var> <input>...)
  string(REGEX REPLACE <match-regex> <replace-expr> <out-var> <input>...)

Manipulation
  string(APPEND <string-var> [<input>...])
  string(PREPEND <string-var> [<input>...])
  string(CONCAT <out-var> [<input>...])
  string(JOIN <glue> <out-var> [<input>...])
  string(TOLOWER <string> <out-var>)
  string(TOUPPER <string> <out-var>)
  string(LENGTH <string> <out-var>)
  string(SUBSTRING <string> <begin> <length> <out-var>)
  string(STRIP <string> <out-var>)
  string(GENEX_STRIP <string> <out-var>)
  string(REPEAT <string> <count> <out-var>)

Comparison
  string(COMPARE <op> <string1> <string2> <out-var>)

Hashing
  string(<HASH> <out-var> <input>)

Generation
  string(ASCII <number>... <out-var>)
  string(HEX <string> <out-var>)
  string(CONFIGURE <string> <out-var> [...])
  string(MAKE_C_IDENTIFIER <string> <out-var>)
  string(RANDOM [<option>...] <out-var>)
  string(TIMESTAMP <out-var> [<format string>] [UTC])
  string(UUID <out-var> ...)

JSON
  string(JSON <out-var> [ERROR_VARIABLE <error-var>]
         {GET | TYPE | LENGTH | REMOVE}
         <json-string> <member|index> [<member|index> ...])
  string(JSON <out-var> [ERROR_VARIABLE <error-var>]
         MEMBER <json-string>
         [<member|index> ...] <index>)
  string(JSON <out-var> [ERROR_VARIABLE <error-var>]
         SET <json-string>
         <member|index> [<member|index> ...] <value>)
  string(JSON <out-var> [ERROR_VARIABLE <error-var>]
         EQUAL <json-string1> <json-string2>)
```
По названиям опций можно понять, что есть функционал поиска, замены, поиск по регулярным фразам, работа с JSON и многое другое

## Работа с интернетом и протоколами
В CMake можно не только пи помощи `file(DOWNLOAD <url> [<file>] [...])` получать из интернета.

#### FetchContent
Этот модуль позволяет заполнять контент во время настройки любым методом, поддерживаемым модулем ExternalProject. В то время как ExternalProject_Add() загружается во время сборки, модуль FetchContent делает контент доступным немедленно, позволяя на этапе настройки использовать контент в таких командах, как операции add_subdirectory(), include() или file().

Подробности заполнения контента следует определять отдельно от команды, выполняющей фактическое заполнение. Такое разделение гарантирует, что все детали зависимостей определены до того, как кто-либо попытается использовать их для заполнения контента. Это особенно важно в более сложных иерархиях проектов, где зависимости могут использоваться несколькими проектами.

Ниже показан типичный пример объявления сведений о содержимом для некоторых зависимостей с последующим их заполнением отдельным вызовом:
```
FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG        703bd9caab50b139428cea1aaff9974ebee5742e # release-1.10.0
)
FetchContent_Declare(
  myCompanyIcons
  URL      https://intranet.mycompany.com/assets/iconset_1.12.tar.gz
  URL_HASH MD5=5588a7b18261c20068beabfb4f530b87
)

FetchContent_MakeAvailable(googletest myCompanyIcons)
```

## Опции
Хотелось бы дать пользователю или разработчику возможность при сборке проекта выбирать те или иные опции проекта. В CMake для такого случая есть специальная команда:
```
option(OPTNAME "Description" DEFAULT)
```
Теперь появляется возможность проверять эту опцию в условиях. А чтобы задать значение опции на этапе конфигурации, надо просто выполнить cmake с -D=VALUE. Например:
`CMakeLists.txt:`
```
option(DEBUG "Enable debug" 0)
```
`shell:`
```shell
cmake -DDEBUG=1 .
```

Ещё пример:  
Функция custom_function содержит вызов команды cmake_parse_arguments, а затем команды печати значений определённых переменных. Далее, функция вызывается с аргументами LOW NUMBER 30 COLORS red green blue, после чего производится печать на экран:
```
function(custom_function)
    # Вызвать механизм обработки аргументов для текущей функции:
    cmake_parse_arguments(CUSTOM_FUNCTION "LOW;HIGH" "NUMBER" "COLORS" ${ARGV})

    # Напечатает "'LOW' = [TRUE]":
    message("'LOW' = [${CUSTOM_FUNCTION_LOW}]")
    #Напечатает "'HIGH' = [FALSE]":
    message("'HIGH' = [${CUSTOM_FUNCTION_HIGH}]")
    # Напечатает "'NUMBER' = [30]":
    message("'NUMBER' = [${CUSTOM_FUNCTION_NUMBER}]")
    # Напечатает "'COLORS' = [red;green;blue]":
    message("'COLORS' = [${CUSTOM_FUNCTION_COLORS}]")
endfunction()

# Вызвать функцию "custom_function" с произвольными аргументами:
custom_function(LOW NUMBER 30 COLORS red green blue)
```

#### Интересный проект на CMake, выводит стрелочные часы
- https://habr.com/ru/articles/253813/
![clock CMake](https://habrastorage.org/r/w1560/files/0f9/b59/429/0f9b59429cb041cca2fe1f101321981c.png)

## Список литературы
-	https://ps-group.github.io/cxx/cmake_cheatsheet
-	https://habr.com/ru/articles/683204/
-	https://cyberleninka.ru/article/n/skriptovye-yazyki-programmirovaniya/viewer
- https://admins.su/znakomstvo-s-cmake-chast-2/
- https://admins.su/znakomstvo-s-cmake-chast-3-cmakecache-moduli-cmake-zavisimosti-sborki/
- https://cmake.org/cmake/help/v2.8.12/cmake.html
- https://cmake.org/cmake/help/latest/command/file.html
- https://runebook.dev/ru/docs/cmake/command/file?page=3
- https://cmake.org/cmake/help/latest/command/string.html
- https://cmake.org/cmake/help/latest/module/FetchContent.html
