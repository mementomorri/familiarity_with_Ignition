# Работа с Ignition

Главный экран имеет следующий вид:

![](https://github.com/mementomorri/Familiarity_with_Ignition/blob/main/Screenshots/1.png)

Главный экран создан с помощью базовых элементов инструмента Symbol factory, который содержит визуальное отображение сути тех процесса, кнопки для управления системой и графики наглядно отображающие историю изменения содержимого емкости в реальном времени и объем откачиваемой из емкости нефти.\
Верхний уровень реализации системы отображает все необходимые по ТЗ элементы: все параметры отображены в инженерных единицах, наглядно отображено состояние технолгического оборудования, реализована возможность менять настроечные параметры системы, аварийные сообщения о неисправности оборудования отображены в виде красных лампочек, которые появляются при аварийных показателях на датчиках, управление оборудованием в виде переключателей, запуск и остановка насосов, открытие и закрытие задвижем, для проверки работы сигнализации загазованности без остановки тех процесса, систему сигнализации можно изолировать от всейостальной системы.\
Система управления объектом имеет следующие тэги:

![](https://github.com/mementomorri/Familiarity_with_Ignition/blob/main/Screenshots/2.png)

Каждый из тэгов расположен в соответствующей его назначению папке, тэг AutoModeOn имеет значение «истина», если система находится в автоматизированном режиме и состояние «ложь» в противном случае. Тэг Tank_level имеет целочисленный формат и отображает текущий уровень заполненности емкости в миллиметрах, он имеет свойство Historian, что означает история его изменений будет записана в таблицу БД SQL сервера развернутого на том же компьютере, что и Ignition. Папка Tank хранит тэги, которые по смыслу связаны со свойствами и состоянием емкости хранящей нефть. Папка Pumps хранит тэги по смыслу связанные с насосами, их состояние (вкл/выкл), показателями с датчиков давления на выкиде, температура подшипников и прочие теги объявляющие время перед аварийным отключением, а также показатели принятые за аварийные. Папка Pinches хранит тэги связанные с задвижками, их состояние (открыта\закрыта), а также теги объявляющие время перед аварийным отключением. Папка GasMeter хранит тэги связанные с датчиком загазованности, показатели аварийного состояния и изолированность датчика. Папка Button_states хранит тэги необходимые для имитирования постоянной подачи нефти в емкость.\
Система управления объектом использует скрипты написанные на ЯП Python версии 2.7, скрипты Gateaway Events имею различные категории. Например, кнопки управления на основном экране имеют встроенные скрипты, которые инициируются при нажатии кнопок, обрабатывают событие и записывают изменения в тэги, а остальные скрипты работают с новыми данными записанными в тэги. Данная конкретная реализация системы управления использует скрипты вызываемые при изменении состояния тэгов:

![](https://github.com/mementomorri/Familiarity_with_Ignition/blob/main/Screenshots/5.png)

И при обнулении таймеров:

![](https://github.com/mementomorri/Familiarity_with_Ignition/blob/main/Screenshots/6.png)

Скрипты типа Tag Change:
* GasMeterAlarm - следит за изменениями в тэге GasMeter, если показатели переданные в тэг превышают первый или второй порог загазованности включает соответствующую сигнализацию;
* PumpsCheck - следит за изменениями в группе тэгов связанных с насосами, при их включении проверяет состояние задвижек, если они закрыты, то автоматически их открывает, если система находится в автоматическом режиме. Если давление после насосов доходит до рабочего, то начинается откачка нефти, таким образом имитируя постепенное повышение оборотов двигателя насоса;
* TankLevelCheck - проверяет изменения уровня емкости, если значение емкости доходит до рабочего, то скрипт переводит насосы во включенное состояние и отключает при достижении нижнего порога. Конкретные пороговые значения задаются на главном экране.\
Скрипты типа Timer:
* GradualPressure - скрипт ежесекундно повышает давление на насосах если они находятся во включенном состоянии, таким образом имитируя постепенное повышение давления на насосах, а также внося случайное повышение или понижение давления, привнося вероятность воссоздать аварийную ситуацию;
* GradualTemperature - постепенно повышает температуру на подшипниках, если насосы находятся во включенном состоянии, таким образом имитируя постепенное повышение температуры на подшипниках, а также внося случайное повышение или понижение температуры, привнося вероятность воссоздать аварийную ситуацию;
* HighOrLowPressureAlarm - скрипт проверяющий состояние насосов, если они включены и вышли за пределы допустимого давления, то сигнализирует об аварийной ситуации и останавливает работу системы;
* IncrDecr - скрипт имитирующий постоянное пополнение емкости нефтью;
* PillowTemperatureAlarm - скрипт проверяющий температуру подшипников, если насосы включены и температуры вышли за пределы допустимого, то сигнализирует об аварийной ситуации и останавливает работу системы;
* PinchAlarm — скрипт проверяет работает ли насос при закрытой задвижке, если задвижка закрыта достаточно долгое время при включенном насосе, то сигнализирует об аварийной ситуации;
* RedundantPumpAutostart - скрипт проверяет включен ли первый насос, например, вручную оператором и включает резервный насос, если содержимое бака на рабочем уровне достаточно долгое время.\
Время, температуры, давление приняты за аварийные можно регулировать на главном экране системы управления. Режим работы системы и её отдельных элементов также можно изменить на главном экране и скрипты учитывают эти факторы.

# Работа с контроллером

При работе с памятью контроллера использовался контроллер H2-DM1E:

![](https://github.com/mementomorri/Familiarity_with_Ignition/blob/main/Screenshots/12.jpg)

В ходе выполнения работы было принято решение перенести тэги и скрипты модуля Gateaway Events в память контроллера, чтобы облегчить работу Ignition и дать ему работать только над верхним уровнем, а непосредственно автоматизированное управление системой предоставить контроллеру.\
Как видно по фото контроллер подключен к рабочему компьютеру через патч-корд, что дает возможность работать через интерфейс Ethernet. В качестве среды разработки была использована программа Do-More, скрипты написанные на языке Python буду перенесены на понятный контроллеру язык релейных диаграмм. Для работы с контроллером был подключен Ignition через веб интерфейс по протоколу Modbus, в архитектуре этого протокола контроллер будет являться клиентом, а Ignition сервером. Modbus имеет стандартизованную систему доступа к данным хранящимся в регистрах четырех типов:\
* 0xxx Coil output - каждая ячейка данного типа регистра хранит один бит, его значение может принять либо 0, либо 1, сюда мы будем записывать тэги типа Boolean;
* 1xxx Coil input - каждая ячейка данного типа регистра хранит один бит, его значение может принять либо 0, либо 1, мы не можем записывать сюда данные при помощи верхнего уровня, только считывать их, вносить же их могу только устройства подключеные в модуль цифрового считывания;
* 3xxx Input register - каждая ячейка данного типа регистра хранит 16 бит, она может принять значение в диапазоне от -32768 до 32768, сюда записываются аналоговые данные с подключеных приборов, мы можем только считывать их. Здесь хранятся целочисленные переменные, одна сюда можно разместить и переменные типа Double, но они займу две ячейки;
* 4xxx Holding register - каждая ячейка данного типа регистра хранит 16 бит, она может принять значение в диапазоне от -32768 до 32768, сюда мы будем записывать тэги типа Intrger.
После подключения контроллера к Ignition (убедиться в подключении контроллера можно в вкладке Status -> Connections -> OPC connections) для тегов нужно изменить место хранения, изначально они хранились во внутренней памяти компьютера, теперь нам нужно указать для них ячейку в соответствующем регистре контроллера, таким образом ещё больше облегчив работу хранения данных для Ignition.

![](https://github.com/mementomorri/Familiarity_with_Ignition/blob/main/Screenshots/11.png)

Пример переноса скриптов в язык релейных диаграмм:

![](https://github.com/mementomorri/Familiarity_with_Ignition/blob/main/Screenshots/13.png)

# Подключение базы данных

В данном проекте было использовано СУБД от Microsoft - SQL Server. БД будет иметь достаточно тривиальный вид:

![](https://github.com/mementomorri/Familiarity_with_Ignition/blob/main/Screenshots/7.png)

Как видно из скриншота, БД имеет следующие таблицы:\
* sqlt_data_1_2021_07 - эта таблица хранит необработанные данные об изменениях в тегах, таблиц такого типа может быть несколько. Первая цифра отвечает за Gateaway от которого БД получает данные, год и месяц периода времени, за который таблица хранит данные;
* sqlth_te - данная таблица хранит метаданные о тэгах, их ID, имя и типы данных;
* sqlth_scinfo - данная таблица хранит информацию о классе scan;
* sqlth_sce - данная таблица хранит данные о начальные и конечные точки во времени для класса scan;
* sqlth_partitions - данна таблица данные о начальной и конечной точке для таблиц типа sqlt_data, разграничивая из временные диапазоны;
* sqlth_drv - данная таблица хранит данные драйверах данных типа historical;
* sqlth_annotations - данная таблица хранит данные о аннотациях касающихся обрабатываемых тэгов.\
В разделе настроек, в подразделе Database откроем вкладку Connections и подключим нашу БД согласно правилам описанным в документации:

![](https://github.com/mementomorri/Familiarity_with_Ignition/blob/main/Screenshots/9.png)

![](https://github.com/mementomorri/Familiarity_with_Ignition/blob/main/Screenshots/10.png)

Добавим некоторым важным, для работы системы, тэгам свойство Historian и проверим работает ли система сохранения истории изменений. Далее представлен результат такой манипцляции в виде запроса к таблице хранящей изменения в тэге tank_value:

![](https://github.com/mementomorri/Familiarity_with_Ignition/blob/main/Screenshots/8.png)

Действительно, Ignition добавил записи об изменениях этих тегов БД, теперь мы можем отобразить эти данные на верхнем уровне работы системы управления, создадим в Ignition пару окон для отображения этих данных в виде графика:


![](https://github.com/mementomorri/Familiarity_with_Ignition/blob/main/Screenshots/3.png)


![](https://github.com/mementomorri/Familiarity_with_Ignition/blob/main/Screenshots/4.png)
