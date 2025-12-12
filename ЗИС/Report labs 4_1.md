<h1 style="text-align:center">Отчет по лабораторной работе №4.1</h1>

<h2>Задание: </h2>
Освоить основные приемы использования программного комплекса SearchInform для перехвата и поиска утечек конфиденциальной информации.

<h2>Выполнение: </h2>

Открыть программу SearchInform AlertCenter и нажать "Запустить сервер"

![Запуск сервера](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/ЗИС%204%20лабораторная/image.png)

И закрыть окно программы через кнопку "Закрыть"
![Окно AlertCenter](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/ЗИС%204%20лабораторная/24.png)

Открыть программу SearchInform NetworkSniffer.
![SearchInform NetworkSniffer Administrator Console](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/ЗИС%204%20лабораторная/25.png)

Перейти во вкладку "фильтры".

![Фильтры SearchInform NetworkSniffer Administrator Console](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/ЗИС%204%20лабораторная/26.png)

И нажать чекбокс "Включить фильтрацию", нажать на кнопку "Добавить". Нажать на добавление пользователя, найти и выбрать пользователя ivanov и нажать на кнопку "Добавить".

![Ivanov](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/ЗИС%204%20лабораторная/27.png)

Нажать на кнопку "Применить изменения" иначе настройка не сохранится.

![Сохранение настройки](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/ЗИС%204%20лабораторная/28.png)

Также, добавить остальных пользователей, такие как: bublik, konev

![Пользователи](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/ЗИС%204%20лабораторная/29.png)

Для того, чтобы удалить пользователя, достаточно нажать на пользователя и выбрать кнопку "Удалить"

![Удаление пользователя](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/ЗИС%204%20лабораторная/30.png)

Для добавления по IP-адресу необходимо выбрать колонку IP адрес и внизу нажать на кнопку "Добавить"

![Добавление IP-адреса](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/ЗИС%204%20лабораторная/31.png)

![Окно с IP-адресом](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/ЗИС%204%20лабораторная/32.png)

Выставить значения и сохранить

![Настройка для IP диапазона](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/ЗИС%204%20лабораторная/33.png)

Далее, необходимо перейти на вкладку "Настройки протоколов" -> HTTP (POST) -> Выбрать пользователя, убрать чекбокс и сохранить настройку

![Настройки протоколов](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/ЗИС%204%20лабораторная/34.png)

Далее переходим на вкладку "Фильтрация SMTP/POP3" и включаем нажатием на чекбокс настройки

![SMTP/POP3](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/ЗИС%204%20лабораторная/35.png)

И выставить следующие настройки:

- Размер (>1000 байт)
- Исключить письмо из перехвата

И нажать на "Применить изменения"

![Перехват письма](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/ЗИС%204%20лабораторная/36.png)

Далее переходим в "Интеграция с почтовыми серверами" -> "Пользователи и фильтрация"

![Пользователи и фильтрация](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/ЗИС%204%20лабораторная/37.png)

Нажать на "Добавить" и указать необходимую почту и нажать "ок"

![Добавление почты](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/ЗИС%204%20лабораторная/38.png)

Добавленая почта отображается в списке. Переходим во вкладку "Определение пользователей" и добавляем маску почт

![Маска почты](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/ЗИС%204%20лабораторная/39.png)

Далее перейти на вкладку "Пользователи" -> "Фильтры" и добавить по следующим параметрам:

- Тема письма
- Исплючить письмо из перехвата

Сохранить настройку

![Маска почты](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/ЗИС%204%20лабораторная/40.png)

Переходим в "Фильтры" -> "Фильтрация HTTP" и выставить фильтр:

- Содержимое ("Vzyatka")
- Исключить документ из перехвата

Нажать на "Применить изменения"

![Взятка](https://raw.githubusercontent.com/sovva6-14/CyberSec/refs/heads/main/Img/ЗИС%204%20лабораторная/41.png)


<h2>Выводы: </h2>

В данной работе была проведена настройка фильтрации по перехвату информации















