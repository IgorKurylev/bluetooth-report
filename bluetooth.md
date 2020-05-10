# Обзор Bluetooth в Android

Пользовательское приложение использует Bluetooth процесс через *Binder* (специальную службу в Android, обеспечивающее межпроцессное взаимодействие). Java Native Interface (JNI - вызов кода на C/C++ из Java) используется Bluetooth процессом для взаимодействия с низкоуревневым Bluetooth стеком. Рисунок 1 объясняет реализацию Bluetooth в Android.

![](https://s09sas.storage.yandex.net/rdisk/fadfe8afdc50881366f5546b9a6a964cacac14000f931503c9d71ef8aa1cf02c/5eb82a4e/VjhUqhsAETQC8mIPmDmVvSsGxaJTp7BGKDTbHC4TO4leCCHblS7Qn-YXyBLZdnhnOFlHAWMwaie0JAfQA6WFBQ==?uid=1040559485&filename=ape_fwk_bluetooth.png&disposition=inline&hash=&limit=0&content_type=image%2Fpng&tknv=v2&owner_uid=1040559485&media_type=image&etag=bb84bb573d3272eb0df245ddbc577700&fsize=40617&hid=f9a1d84542715f3826d6e1403a4ebc9b&rtoken=XNkjP4EiD0zY&force_default=yes&ycrid=na-313dd79bc5ac172a0d535744f6eb6626-downloader1f&ts=5a54da384af80&s=43db5e9ce00f56fb45e5890388711a7a40705a98c9a45cce792d8a0ace8de97a&pb=U2FsdGVkX18Y5U8jILJrM3FKBtjpKzpHodyIVUm9s4yeWe20pyWP9_87M2vW62eMiw7c4Rsna6SyOONQpbWrngDkwLb3kmfKywWn6ubcwNo)
***Рисунок 1*** архитектура Bluetooth в Android
     
- **Application framework** - пользовательское приложение, использующее библиотеку *android.bluetooth* и механизм *Binder*
- **Bluetooth system service** - служба Bluetooth, расположенная в *packages/apps/Bluetooth* как отдельное приложение. Взаимодействует с уровнем **HAL** через **JNI**
- **JNI** - код на С/C++, расположенный в *packages/apps/Bluetooth/jni* (как раз представляет собой библиотеку *android.bluetooth*). Когда выполняется какая-либо Bluetooth операция, например обнаруживается новое Bluetooth устройство,  ожидает выполнения специальных функций-коллбэков из уровня **HAL**
- **Hardware abstraction level (HAL)** - более близкий к аппаратной части уровень. Предназначен для связи ядра Android c аппаратной частью Bluetooth стека. Реализовывает специальный интерфейс, который используется *android.bluetooth*.
- **Bluetooth stack** - реализация стека протоколов Bluetooth, расположен в *system/bt*. Расширяет уровень **HAL** дополнительными модулями и конфигурациями.
- **Vendor extensions** - может быть добавлен вендором (компанией, которая выпускает конкретный девайс) для реализации командного интерфейса контроллера (драйвера **HCI**). Это позволит получить доступ к парамерам настройки Baseband (часть системы Bluetooth реализующая функции обработки сигнала на физическом уровне)
