# Логирование в стиле log4j

Библиотека предназначена для удобного вывода сообщений в привязке к их "уровням важности"
 
Функционал logos не повторяет полностью log4j, а, скорее, берет из него какие-то аспекты поведения.

Библиотека портирована на платформу 1С из пакета https://github.com/oscript-library/logos

старый адрес - https://github.com/silverbulleters/logos-1c

# Использование

## Именованные журналы сообщений (Логи)

Каждый лог имеет собственное имя, по которому он может быть идентифицирован. Имена логов глобально видимы и любой из них может быть получен по имени лога.

Для того, чтобы начать логирование, требуется вызвать метод `ПолучитьЛог` модуля `Логирование`

    Лог = Логирование.ПолучитьЛог("oscript.app.messages");
    
### Соглашение об именовании

Принята следующая схема именования логов:

    область.класс_приложения.имя_лога
    
* Область - это произвольное имя, определяющее некую совокупность возможных логов. Часто, в качестве области используется имя поставщика приложения, либо имя набора библиотек. Для всех пакетов, входящих в oscript-library используется имя области "oscript". Приложения, создаваемые, например, в рамках гипотетического проекта "Аврора" могут использовать область "aurora".
* Класс_приложения - это разделитель на тип модуля - библиотека, или приложение. Например, библиотечный пакет "cmdline" использует класс `lib`, а консольное приложение "gitsync" - класс 'app'
* Имя_лога - это, собственно, идентификатор журнала.

### Примечание

В отличие от log4j логи в logos не являются иерархическими, но в перспективе могут стать таковыми.

## Уровни логирования

Существует 5 уровней важности сообщений:

* Отладка
* Информация
* Предупреждение
* Ошибка
* Критичная ошибка

Для каждого из уровней logos предоставляет отдельный метод, который выводит сообщение с данным уровнем важности. В процессе эксплуатации приложения для каждого из логов можно устанавливать уровень выводимых сообщений. Например, если установлен уровень "Предупреждение", то выводиться будут только предупреждения или более важные сообщения (Ошибка, КритичнаяОшибка)

    Лог.УстановитьУровень(УровниЛога.Отладка) // выводить все, в т.ч. отладочные сообщения
    
По умолчанию установлен уровень "Информация".

### Пример использования

    Лог = Логирование.ПолучитьЛог("oscript.test.levels");
    Лог.УстановитьУровень(УровниЛога.Ошибка);
    
    Лог.Отладка("Переменная А = 7");
    Лог.Информация("Переменная А = 7");    
    Лог.Ошибка("Неверно указан путь к файлу!");
    
    ПеремА = "А";
    Лог.Отладка("Переменная %1 = %2", ПеремА, 7);
    Лог.Информация("Переменная %1 = %2", ПеремА, 7);
    Лог.Ошибка("Неверно указан путь к файлу %1!", ПеремА);
    
## Способы вывода (appenders)

Приложение выводит сообщения в лог, а сам лог может выводиться в произвольное место. За конкретную реализацию вывода отвечает *Способ вывода*. В Log4j это называется appender.

В составе logos поставляется 2 способа вывода - в консоль и в файл.
Причем, способы вывода - это список, т.е. лог может писаться в несколько мест одновременно.

    ФайлЖурнала = Новый ВыводЛогаВФайл
    ФайлЖурнала.ОткрытьФайл("/var/log/oscript.test.log");
    Лог.ДобавитьСпособВывода(ФайлЖурнала);
    Лог.Информация("Это строка лога");
    Лог.Закрыть(); // при включении логирования в файл рекомендуется закрывать лог.

### Особенности добавления Способов вывода

При создании лога в него автоматически будет добавлен способ вывода *ВыводЛогаВКонсоль*. Однако, если вручную в лог будет добавлен другой способ вывода, то "автоспособ" будет удален. Иными словами, пример кода выше будет писать только в файл, т.к. консольный вывод при ручном добавлении способа вывода будет отключен.

## Форматирование сообщений

За форматирование выводимых сообщений отвечает *Раскладка*. Раскладка по умолчанию форматирует сообщения следующим образом:

    УРОВЕНЬСООБЩЕНИЯ - ТекстСообщения
    
Раскладка - это класс, реализующий метод *Форматировать(Знач Уровень, Знач Сообщение)*. Установить собственную раскладку можно методом "УстановитьРаскладку".

    Функция Форматировать(Знач Уровень, Знач Сообщение) Экспорт

        Возврат СтрШаблон("%1: %2 - %3", ТекущаяДата(), УровниЛога.НаименованиеУровня(Уровень), Сообщение);
    
    КонецФункции
    
    Лог.УстановитьРаскладку(ЭтотОбъект);
    
В приведенной раскладке в сообщение помимо уровня и текста будет добавлено текущая дата.

# Конфигурирование логов

Предусмотрено изменение параметров логирования через специальный файл конфигурации, либо через переменную окружения.
Независимо от способа задания настроек, формат настроек остается одинаковым.

Файл конфигурации должен лежать рядом со стартовым скриптом (см. метод *СтартовыйСценарий()*) и называться *logos.cfg*. Кроме того, настройки можно задать через переменную окружения LOGOS_CONFIG. Из-за неудобства ввода перевода строк в переменных окружения - в данном случае настройки надо разделять не переводом строк, а точкой-с-запятой. Формат настроек идентичен, независимо от способа задания конфига - в файле или в переменной.

Следует учитывать, что переменная окружения имеет приоритет над файлом, и если задана она, то настройки берутся из нее, файл игнорируется.

### Пример задания настроек через командную строку:
```cmd
set LOGOS_CONFIG=logger.oscript.lib.commands=DEBUG;logger.oscript.lib.cmdline=DEBUG
```

## Формат настроек логирования

Каждая настройка задается в виде пары ```ключ=значение```. Причем, ключ является составным. Компоненты ключа разделены точками.
Первым компонентом ключа идет *класс настройки*. Предусмотрены 2 класса - *logger* и *appender*. Первый отвечает за настройку конкретного журнала. Второй - за настройку способов вывода, используемых журналами.

### Настройка журнала (класс logger)

    logger.имя_журнала=Уровень[,СпособыВывода]

* имя_журнала - это имя лога, как он описан в вызове метода ```Логирование.ПолучитьЛог();```, например, **oscript.lib.v8runner**.
* СпособыВывода - это список произвольных имен способов вывода, которые привязаны к журналу. Конкретная настройка каждого из заявленных способов вывода выполняется в классе настроек *appender*.

Например, журнал **oscript.lib.v8runner** может быть настроен следующим образом:

    logger.oscript.lib.v8runner=DEBUG, v8rdebug, console

### Корневой журнал (логгер)

В классе настроек logger возможно указать специализированное имя ``rootLogger``. Настройки корневого логгера влияют на все прочие журналы. Это удобно, если вы хотите просто включить отладку по всем журналам или все журналы направить в файл.

Например:

    logger.rootLogger=DEBUG

### Настройка способа вывода (класс appender)

    logger.oscript.lib.v8runner=DEBUG, v8rdebug, console
    appender.v8rdebug=ВыводЛогаВФайл
    appender.v8rdebug.file=/var/log/v8runner-debug.log
    
    appender.console=ВыводЛогаВКонсоль

В приведенном примере для лога oscript.lib.v8runner установлен уровень Отладка и заявлено 2 способа вывода. Они названы v8rdebug и console (названия произвольные).

Далее, в конфигурации указаны настройки заявленных способов вывода (аппендеров). Обязательным параметром является класс (тип языка), реализующий способ вывода. В данном примере указаны типы **ВыводЛогаВФайл** и **ВыводЛогаВКонсоль**.

    // формат указания класса реализации
    appender.имя_способа_вывода=Класс

Для каждого класса могут потребоваться какие-то свои параметры. Например, класс ВыводЛогаВФайл требует задания свойства file. Свойства способа вывода задаются через точку от имени способа вывода:

    // формат указания свойства
    appender.имя_способа_вывода.свойство=значение

# Примеры

## Настройка лога в шаге фичи
```bsl
&НаСервере
Процедура ПроверитьАвторизацию_Сервер(Путь, Логин, Пароль) Экспорт

	НастройкиЛогирования = Обработки.НастройкиЛогированияЛог.Создать();
	НастройкиЛогирования.ЗаписатьВКонфигурацию("Модуль1САПИ", НастройкиЛогирования.УровниЛога.Отладка);

	//Обычный вызов боевого кода, у которого где-то внутри используется Лог.Отладка, Информация и прочее - см.ниже

КонецПроцедуры
```

или дополнительный вывод сообщений в ЖР
```bsl
Процедура ВключитьОтладкуЛогаХранилища()

	ПараметрыВыводаВЛог = СтрШаблон("DEBUG, v8reg, console
	|appender.v8reg=ВыводЛогаВЖР
	|appender.v8reg.ИмяСобытия=%1
	|appender.v8reg.Уровень=Предупреждение
	|
	|appender.console=ВыводЛогаВКонсоль", "Событие1.Подвид1");

	НастройкиЛогирования = Обработки.НастройкиЛогированияЛог.Создать();
	НастройкиЛогирования.ЗаписатьВКонфигурацию("Модуль1САПИ", НастройкиЛогирования.УровниЛога.Отладка, ПараметрыВыводаВЛог);

КонецПроцедуры
```

## Создание лога в боевом коде
```bsl
МенеджерЛогирования = Обработки.МенеджерЛогированияЛог.Создать();
Лог = МенеджерЛогирования.ПолучитьЛог("Модуль1САПИ");
```

## Обычный вызов внутри боевого кода

```bsl
Лог.Информация("Создаю пользователя ЛогинАвторизации <%1>", ЛогинАдминистратора);
Лог.Отладка("СоздатьПользователя - ЛогинАвторизации <%1>", ЛогинАдминистратора);
```
