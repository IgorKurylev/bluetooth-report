# Введение
TODO: types of vulnerabilities, popular vulnerabilities, impact on android

**Remote control execution (RCE)** - уязвимость, при которой происходит удаленное выполнение кода на взламываемом компьютере или устройстве. Является одной из самых опасных уязвимостей.

# 1.1 Краткое введение в Bluetooth
-------------------------------
Многие сетевые протоколы предоставляют каналы связи для коммуникации и оставляют их использование на усмотрение пользователей. В противоположность этому Bluetooth предоставляет специальные приложения - **профили (profiles)**. И для каждого из них предоставляет свой набор протоколов. Это очень усложняет архитектуру Bluetooth ввиду того, что всего профилей более 30.

Как показано на рисунке 1.1 вся структура Bluetooth протоколов является некой альтернативой стека TCP/IP, начиная с физического уровня и заканчивая прикладным, следуя при этом модели OSI.

![](https://s88sas.storage.yandex.net/rdisk/91e7adde4126c91bcac81841a93ce640228d6370aa6f4912925e675e083279e5/5eb84a17/VjhUqhsAETQC8mIPmDmVva-EHqaGqYTZLIiZqGPMbBTu1OXbNhCL8lhGAkqYB8-2rysZQ-1OwEyosfb09U5XYQ==?uid=1040559485&filename=image--015.jpg&disposition=inline&hash=&limit=0&content_type=image%2Fjpeg&tknv=v2&owner_uid=1040559485&etag=890f4d45673b4df6e94ea7e944c6caa0&hid=73980904cc19cd0794eb3ce0599c8b0a&media_type=image&fsize=67402&rtoken=XEck4RTJReva&force_default=yes&ycrid=na-6ae9d863a307be80c70f6a295e18a449-downloader16h&ts=5a54f888573c0&s=3388ba59170a6fa32a2cd5c61efbc66565896f11164e782cf8eb44ba4f3c18a2&pb=U2FsdGVkX19uyND8AKNT19OvG5LC0P1xTDO9eBFP2sfVKr3GKynq7WFq0o-SF_TYr6XAQQ2Txw3vfNWJhg7jhBm0itWiyM8x3fGfve7kZ_Q)
**Рисунок 1.1.1** архитектура стека Bluetooth протоколов

Чем ниже протокол - тем ближе он расположен к физическому уровню. Самые нижние уровни реализованы на контроллерах Bluetooth. Эти чипы взаимодействуют с ОС через интерфейс контроллера (**HCI**). Все протоколы выше **HCI** (например *L2CAP*, *SMP*, *SDP*) реализованы на уровне ОС и входят в отдельный стек Bluetooth конкретной ОС (иногда туда может входить и **HCI**). Профили на рисунке 1.1.1 представлены белым цветом и могут использовать часть стека протоколов для своих целей. Так как каждая ОС использует только собственный стек протоколов Bluetooth, то любая уязвимость, найденная в этом стеке затрагивает все устройства с этой ОС. Например, Linux и ранние версии Android используют стек *BlueZ*. Начиная с версии 4.2 Android использует свой собственный стек *Bluedroid*.

**BlueBorne** представляет собой несколько уязвимостей разных ОС на отдельных уровнях иерархии Bluetooth. В своем докладе я затрагиваю уязвимости Android и Linux (так как *BlueZ* ранее использовался в Android). На рисунке 1.1.2 показаны уровни, на которых обнаружены данные уязвимости.

![](https://s172i.storage.yandex.net/rdisk/05ad1ce0b74c4274a2e32bdd30f48b54aa190e9650041d7267617f248a7158af/5eb85713/VjhUqhsAETQC8mIPmDmVvSVe9WY5_j_kRagJoJ8NNGMqtSBNjfxEEXoNbYu-sAXjOu56ky8T7tmtNhLFSuuKsA==?uid=1040559485&filename=image--013.jpg&disposition=inline&hash=&limit=0&content_type=image%2Fjpeg&tknv=v2&owner_uid=1040559485&fsize=112968&hid=e954b81d4140c54053a026302e433d39&etag=6432896b8aa4b7b82fc66ba09f40eacc&media_type=image&rtoken=ENNuB1zDTHCj&force_default=yes&ycrid=na-cc857b24fee1f7c86b44cc4528fd1b25-downloader18f&ts=5a5504ea5aac0&s=e52b64da1e5f7205be82446401bd4e107ebc1fae7d4c843cf2af1097f499bc18&pb=U2FsdGVkX19ZZOHpwZsIA-X_2KnPb3jW_y04TIdMKKfmEP-SjN_Sc_qJjQjVNpRwR5C3ToiFjfVT17q4623pTa6uycX56ghkWg8OxzV0S_E)
**Рисунок 1.1.2** уровни стека Bluetooth, в которых найдены уязвимости

Кратко о каждом затронутом уровне:
- **L2CAP** - протокол транспортного уровня. Обеспечивает согласование **MTU** (максимальный размер полезного блока данных одного пакета), порядка следования пакетов и, при необходимости, надежность доставки.
- **SDP** - протокол обнаружения сервисов и приложений, которые поддерживаются Bluetooth устройством.
- **BNEP** - протокол инкапсуляции (обычно IP пакетов) поверх Bluetooth. Может, например, использоваться при раздаче интернета через Bluetooth.
- **PAN** - профиль, который использует **BNEP** для создания IP сетей поверх Bluetooth.

Более подробно о каждом уровне в разделах, описывающих уязвимости. 

# 1.2 Обзор Bluetooth стека в Android
--------------------------------
Пользовательское приложение использует Bluetooth процесс через *Binder* (специальную службу в Android, обеспечивающую межпроцессное взаимодействие). Java Native Interface (**JNI** - вызов кода на C/C++ из Java) используется Bluetooth процессом для взаимодействия с низкоуревневым Bluetooth стеком. Рисунок 1 объясняет реализацию Bluetooth в Android.

![](https://s09sas.storage.yandex.net/rdisk/fadfe8afdc50881366f5546b9a6a964cacac14000f931503c9d71ef8aa1cf02c/5eb82a4e/VjhUqhsAETQC8mIPmDmVvSsGxaJTp7BGKDTbHC4TO4leCCHblS7Qn-YXyBLZdnhnOFlHAWMwaie0JAfQA6WFBQ==?uid=1040559485&filename=ape_fwk_bluetooth.png&disposition=inline&hash=&limit=0&content_type=image%2Fpng&tknv=v2&owner_uid=1040559485&media_type=image&etag=bb84bb573d3272eb0df245ddbc577700&fsize=40617&hid=f9a1d84542715f3826d6e1403a4ebc9b&rtoken=XNkjP4EiD0zY&force_default=yes&ycrid=na-313dd79bc5ac172a0d535744f6eb6626-downloader1f&ts=5a54da384af80&s=43db5e9ce00f56fb45e5890388711a7a40705a98c9a45cce792d8a0ace8de97a&pb=U2FsdGVkX18Y5U8jILJrM3FKBtjpKzpHodyIVUm9s4yeWe20pyWP9_87M2vW62eMiw7c4Rsna6SyOONQpbWrngDkwLb3kmfKywWn6ubcwNo)

***Рисунок 1.2.1*** архитектура Bluetooth стека в Android
     
- **Application framework** - пользовательское приложение, использующее библиотеку *android.bluetooth* и механизм *Binder*
- **Bluetooth system service** - служба Bluetooth, расположенная в *packages/apps/Bluetooth* как отдельное приложение. Взаимодействует с уровнем **HAL** через **JNI**
- **JNI** - код на С/C++, расположенный в *packages/apps/Bluetooth/jni* (как раз представляет собой библиотеку *android.bluetooth*). Когда выполняется какая-либо Bluetooth операция, например обнаруживается новое Bluetooth устройство,  ожидает выполнения специальных функций-коллбэков из уровня **HAL**
- **Hardware abstraction level (HAL)** - более близкий к аппаратной части уровень. Предназначен для связи ядра Android c аппаратной частью Bluetooth стека. Реализовывает специальный интерфейс, который используется *android.bluetooth*.
- **Bluetooth stack** - реализация стека протоколов Bluetooth, расположен в *system/bt*. Расширяет уровень **HAL** дополнительными модулями и конфигурациями.
- **Vendor extensions** - может быть добавлен вендором (компанией, которая выпускает конкретный девайс) для реализации командного интерфейса контроллера (драйвера **HCI**).

# 2.1 L2CAP и уязвимость BlueZ стека Linux
----------------------------------------
### Обзор L2CAP

На стороне ОС это самый нижний уровень иерархии Bluetooth. L2CAP отвечает за связь между различными сервисами (протоколами) Bluetooth стека. Инкапсулирован в транспортный протокол **ACL** (предоставляет **chanel IDs**, далее **CIDs**, аналоги портов в UDP или TCP, куда передаются пакеты, но не предоставляет надежности доставки). Спецификация Bluetooth может резервировать некоторые **CIDs** для своих целей. Другие **CIDs** присваиваются динамически. Например, если некоторая конечная точка хочет послать сообщение какому-то Bluetooth сервису, ей будет динамически выделен свой **CID**.

Во время создания нового L2CAP соединения, две конечные точки пытаются достичь согласования (по пропускной способности и иным параметрам), обмениваясь специальными пакетами, **запросом** и **ответом** конфигурации. Запрос конфигурации содержит  параметры, по которым можно сказать о типе используемого соединения.

### Процесс обмена конфигурациями

В документации Bluetooth запросы и ответы конфигурации обозначены как **L2CAP_ConfReq** и **L2CAP_ConfResp** сообщения. Начальный обмен этими сообщениями между конечными точками называется **инициализирующим рукопожатием**. *L2CAP_ConfResp* содержит статус-код, который информирует отправителя о том, приняты ли его параметры конфигурации, или в этом отказано. 
![](https://s426sas.storage.yandex.net/rdisk/7ba712398f35527520cf58e60a6a382bf88d951b8cd07ad0fe6826a846303493/5eb9478f/VjhUqhsAETQC8mIPmDmVvSyhKRGbhB5iP6-Vw5bQ7hnQLdPFg_SR_W3tLW7L1znlpRccNWJUJhPkSejmCSMxSA==?uid=1040559485&filename=image--017.jpg&disposition=inline&hash=&limit=0&content_type=image%2Fjpeg&tknv=v2&owner_uid=1040559485&media_type=image&etag=a9eff1fa0e3b223ff82729fa892595ff&fsize=83232&hid=7b142acf5260421972b0b442be49b52c&rtoken=duONR8jL8QxC&force_default=yes&ycrid=na-ab47018f1d12713e46066aa1de4d1d27-downloader12h&ts=5a55ea4167f80&s=775d4364d6d747aca4be109a04e712e5133f0b577db220b0571c26beddff28d2&pb=U2FsdGVkX1-vkxxJSI8N8IOLbE9Pf3tvA--6-LLx1rCDdvFtKsZ55TDwkOyViBA_4R_Pgu47gdzRou2U9st_Wn4NFYhARDiGDwu1xSyFFwM)
**Рисунок 2.1.1** типичный процесс обмена конфигурацией. В данном примере две конечные точки обмениваются информацией об **MTU** (максимальный размер полезного блока данных одного пакета). Остальные параметры выставлены по умолчанию.

Рисунок 2.1.1 иллюстрирует процесс обмена конфигурациями. Девайс **А** запрашивает **MTU** как параметр (*option = 0x01*) и его величину *0x100*, которую девайс **B** принимает и затем запрашивает величину **MTU** как *0x200*, которую девайс **A** также принимает. Таким образом, максимальная величина сообщения, которое устройство **А** может послать устройству **B** составляет *0x100* и обратно от **B** к **A** *0x200*. 

В отличие от вышеприведенного примера, устройство может отклонить параметры, которые ему не подходят. Чтобы облегчить дальнейший обмен конфигурациями, его ответ может содержать альтернативные, приемлимые для себя параметры. Следующая выдержка кода из ядра Linux отражает этот момент. Изначально **MTU** инициализировано по умолчанию.
```c++
// если запрошенный MTU меньше 
// чем минимальный по умолчанию
if (mtu < L2CAP_DEFAULT_MIN_MTU)
    // отбрасываем
	result = L2CAP_CONF_UNACCEPT;
else {
    // присваиваем MTU каналу
	chan->omtu = mtu;
	// заканчиваем настройку MTU
	set_bit(CONF_MTU_DONE, &chan->conf_state);
}
// добавляем настройки конфигурации
l2cap_add_conf_opt(&ptr, L2CAP_CONF_MTU, 2, chan->omtu, endptr - ptr);
```
Выдержка из исходного кода функции *l2cap_parse_conf_req* (*net/bluetooth/l2cap_core.c*)

Таким образом, если предложенное значение **MTU** нам не подходит, в настройки конфигурации войдет значение по умолчанию (мы предложим свое значение). Таким образом процесс обмена конфигурациями продолжится, пока стороны не придут к взаимовыгодным параметрам соединения.

Существует иной механизм процесса обмена конфигурациями для более отказоустойчивого соединения под названием **Extended Flow Specification (EFS)**. Все **EFS** параметры должны быть перепроверены каждой конечной точкой соединения. Поэтому ответ от устройства на конфигурационный запрос может быть **"в ожидании" (pending)**, так как оно не до конца проверило все **EFS** параметры, что будет использовано в уязвимости. После того, как устройства обменялись всеми **EFS** параметрами, стороны достигнут взаимного соглашения.

### RCE уязвимость ядра Linux (CVE-2017-1000251) 

Данная уязвимость затрагивает BlueZ стек ядра Linux, а конкретно реализацию механизма **EFS** протокола L2CAP в функции *l2cap_parse_conf_rsp*. 
Первая ее часть приведена в следующей выдержке.
```c++
static int l2cap_parse_conf_rsp(struct l2cap_chan *chan, void *rsp, int len,
void *data, u16 *result)
{
    // rsp - указатель на буфер в котором 
    // находится конфигурационный ответ
    // len - его длина
    // data - указатель на буфер
    // в который помещаются параметры после проверки
    struct l2cap_conf_req *req = data;
    void *ptr = req->data;
    // ...
    while (len >= L2CAP_CONF_OPT_SIZE) {
        // получаем в цикле элементы из буфера rsp
        len -= l2cap_get_conf_opt(&rsp, &type, &olen, &val);
        switch (type) {
            case L2CAP_CONF_MTU:
                // проверяем MTU
                l2cap_add_conf_opt(&ptr, L2CAP_CONF_MTU, 2, chan->imtu);
                break;
            case L2CAP_CONF_FLUSH_TO:
                chan->flush_to = val;
                l2cap_add_conf_opt(&ptr, L2CAP_CONF_FLUSH_TO, 2, chan->flush_to);
                break;
            // и другие параметры
        }
    }
    // ...
    return ptr - data;
}
```
Выдержка из исходного кода функции *l2cap_parse_conf_rsp* (*net/bluetooth/l2cap_core.c*)

Функция получает конфигурационный ответ в буфере на который указывает ***rsp*** и в цикле получает по одному каждый конфигурационный параметр, используя **l2cap_get_conf_opt**. Каждый полученный элемент проверяется и записывается обратно через указатель ***ptr*** (поле *data* в структуре *l2cap_conf_req*) в буфер ответа, на который указывает ***data***. Здесь важно заметить, что длина буфера ***data*** в функцию не передается.

Размер приходящего ответа никак не ограничен, что позволяет атакующей стороне прислать ответ ***rsp*** в котором могут быть дупликаты. Как результат, все содержимое ***rsp*** (включая возможные дупликаты) скопируются в буфер ***data***. Функция в которой формируется ***data*** (**l2cap_parse_conf_rsp**) вызывается из двух мест в **l2cap_config_rsp**, которая обрабатывает сообщения ответов конфигурации. Эти два места похожи, поэтому оба могут использоваться при эксплуатации уязвимости. Первый фрагмент такого участка представлен на следующей выдержке исходного кода.

```c++
switch (result) {
    case L2CAP_CONF_SUCCESS:
        ...
        break;
    case L2CAP_CONF_PENDING:
        // если устройство в состоянии "ожидание"
        set_bit(CONF_REM_CONF_PEND, &chan->conf_state);
        if (test_bit(CONF_LOC_CONF_PEND, &chan->conf_state)) {
            char buf[64];
            // buf передается как буфер data
            // rsp->data в качестве rsp
            len = l2cap_parse_conf_rsp(chan, rsp->data, len, buf, &result);
        ...
        goto done;
```
Выдержка из исходного кода функции *l2cap_config_rsp* (*net/bluetooth/l2cap_core.c*)

*switch* проверяет значение, которое было было получено из ответа конфигурации, что может управляться атакующим. Далее роль ***data*** будет играть ***char buf[64]***. Данный участок кода исполнится только если устройство будет в состоянии "в ожидании", что может спровоцировано следующим фрагментом кода.
```c++
if (remote_efs) {
    if (chan->local_stype != L2CAP_SERV_NOTRAFIC &&
        efs.stype != L2CAP_SERV_NOTRAFIC &&
        efs.stype != chan->local_stype) {
    ...// We don’t want this branch, easy to avoid
    } else {
        /* Send PENDING Conf Rsp */
        result = L2CAP_CONF_PENDING;
        set_bit(CONF_LOC_CONF_PEND, &chan->conf_state);
    }
}
```
