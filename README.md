## Настройка уборки по комнатам на пылесосе Xiaomi Mijia Robot Vacuum Mop 3C (ijai.vacuum.v18)

### Сначала нам необходимо скачать и установить на андроид смартфон приложение [MiHome от Vevs](https://rumihome.ru/prilozheniya/mihome-vevs)

Качаем и устанавливаем приложение. После подключаем туда наш пылесос. По умолчанию плагин пылесоса на китайском языке, поэтому рекомендую сменить язык приложения на английский. После подключения строим в приложении карту помещения, если нужно разделяем комнаты и даем им имена в разделе "Area editor". 

Далее в приложении нужно включить запись логов. Эта настройка находится в разделе "Experemental features" - "Write custom log file" в настройках самого приложения (не плагина). Сами логи будут писать по следующему пути в памяти смартфона "/storage/emulated/0/vevs/logs/miio/[цифры].txt". Это в принципи корень пользовательской файловой системы при использовании большинства проводников.

Далее переходим в плагин пылесоса, выбираем одну или несколько комнат, запускаем уборку. После запуска идем к файлу логов, забираем его, открываем и смотрим. Нам нужны строки, которые отвечают за получение идентификаторов комнат. В моем случае строка выглядела так:

`{"params":{"aiid":18,"did":"1037735403","in":[1667588145,"[[\"110001095836\",11],[\"110001093812\",10],[\"110001095840\",12],[\"110001095837\",13]]"],"siid":10}} /miotspec/action`

Из этой строки нам интересна эта строка `[\"110001095836\",11],[\"110001093812\",10],[\"110001095840\",12],[\"110001095837\",13]`, это и есть те самые идентификаторы и номера комнат, которые нам позже понадобятся.

Помимо идентификаторов комнат на следующем эиапе нам понадобится токен пылесоса, его можно получить здесь же в MiHome в плагине пылесоса в пункте "Additional settings - Network info - Token". Берем токен, позже он нам понадобится.

### Идем в Home Assistant
В HomeAssistant пылесос необходимо подключить с помощью интеграции [Xiaomi Miot Auto](https://github.com/al-one/hass-xiaomi-miot). Установить проще всего через HACS.

Устанавливаем интеграцию, через GUI HA идем в Настройки - Устройства и службы - Добавить интеграцию. Находим там установленную ранее интеграцию, выбираем метод подключения "Add device using host/token" (чтобы не зависеть от облака) и вводим IP пылесоса (предварительно задав статический адрес в маршрутизаторе) и токен, полученный ранее.

**Пылесос должен залететь в HA с сопутствующими ему сенсорами и инпутами.**

### Непосредственно запуск зональной уборки

Вообще, использую эту интеграцию можно с пылесосом делать практически что угодно, что можно из MiHome, правда придется копаться в методах и параметрах API, которые опубликованы для нашего пылесоса здесь - [ijai.vacuum.v18](https://home.miot-spec.com/spec/ijai.vacuum.v18). Мы же остановимя на том, как запустить уборку определенной комнаты.

Открываем панель разработчика в HA, переходим на вкладу Службы и находим нужную нам службу `Xiaomi Miot Auto: call_action`. В качестве **entity_id** указываем основной объект пылесоса (домен **vacuum**), **siid: 2 / aiid: 7** (что соответствует вызову действия уборки комнаты), в **params** передаем идентфиикатор одной из комнат, полученный ранее в виде строки, так же включаем параметр **throw**, чтобы получить ответ от пылесоса в уведомлениях HA. На выходе в yaml будет примерно так:

```
service: xiaomi_miot.call_action
data:
  entity_id: vacuum.ijai_v18_b713_robot_cleaner
  siid: 2
  aiid: 7
  throw: true
  params: "110001095836"
```
Запускаем службу, смотрим, что в уведомлениях HA появляется успешный ответ от пылесоса `{'code': 0}` и собственно пылесос едет убираться в какую-то из ваших комнат.

### Сопоставление комнат

Далее дело за простым, отправляем так пылесос в каждую из комнат, идентификаторы которых вы получили, и сопоставляем идентификатор комнате. Я себе записал так:
```
"110001095836" - ванная
"110001093812" - спальня
"110001095840" - гостиная
"110001095837" - кухня
```

Теперь нет труда наделать скриптов с запуском службы с указанием соответствующей комнаты и запускать как и когда вам нужно.

### Дополнения

Вместе с пылесосом в HA должен прилететь input_select с выбором вида уборки (Влажная/Сухая/Влажная и сухая). Можно запускать скрипт уборки предварительно выбрав сам вид уборки инпутом, поэтому можно запустить отдельно влажную, отдельно сухую уборку любой из комнат.

Жаль, но пока не нашел как в HA интегрировать карту уборки. В интеграции, которая это умеет для пылесосов Xioami пока нет поддержки устройств производства ijai. Но, уже есть запрос на реализацию, можете присоедениться к [https://github.com/PiotrMachowski/Home-Assistant-custom-components-Xiaomi-Cloud-Map-Extractor/issues/200](Issue) и тоже ждать =)

### Рекомендации

Для отображения пылесоса в графическом интерфейсе рекомендую карточку [Vacuum card](https://github.com/denysdovhan/vacuum-card)

Для запуска зональной уборки голосом через Алису рекомендую интеграцию AlexxIT [VacuumZones](https://github.com/AlexxIT/VacuumZones)


