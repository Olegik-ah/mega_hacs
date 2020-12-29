# MegaD HomeAssistant custom component

Интеграция с [MegaD-2561](https://www.ab-log.ru/smart-house/ethernet/megad-2561)

## Основные особенности:
- Настройка как из yaml так и из веб-интерфейса
- При настройки из веба все порты автоматически добавляются как устройства (для обычных релейных выходов создается 
  `light`, для шим - `light` с поддержкой яркости, для цифровых входов `binary_sensor`, для температурных датчиков
  `sensor`)
- Возможность работы с несколькими megad
- Обратная связь по mqtt
- Команды выполняются друг за другом без конкурентного доступа к ресурсам megad
## Устройства
Поддерживаются устройства: light, switch, binary_sensor, sensor. light может работать как диммер
## Установка
Рекомендованнй способ - через [HACS](https://hacs.xyz/docs/installation/installation).
После установки HACS, нужно перейти в меню hacs -> integrations, далее в верхнем правом углу
нажать три точки, где будет `Custom repositories`, открыть, нажать add и добавить `https://github.com/andvikt/mega_hacs.git`

Альтернативный способ установки:
```shell
# из папки с конфигом
wget -q -O - https://raw.githubusercontent.com/andvikt/mega_hacs/master/install.sh | bash -
```
Перезагрузить HA

Для обновления повторить
## Зависимости
Перед использованием необходимо настроить интеграцию mqtt в HomeAssistant

## Настройка из веб-интерфейса
`Настройки` -> `Интеграции` -> `Добавить интеграцию` в поиске ищем mega

## Пример настройки с помощью yaml:
```yaml
mega: 
  mega1:
    host: 192.168.0.14
    name: hello
    password: sec
    mqtt_id: mega # это id в конфиге меги

light:
  - platform: mega
    mega1:
      switch:
        - 1 # можно просто перечислить порты
        - 2
        - 3
      dimmer:
        - port: 7
          name: hello # можно использовать расширенный вариант с названиями
        - 9
        - 10

binary_sensor:
  - platform: mega
    mega1:
      - port: 16
        name: sensor1
      - port: 18
        name: sensor2

sensor:
  - platform: mega
    mega1:
      - port: 10
        name: some temp
        type: w1
        key: temp
      - port: 10
        name: some hum
        type: w1
        key: hum

switch:
  - platform: mega
    mega1:
      - 11

```

## Сервисы
```yaml
save:
  description: Сохраняет текущее состояние портов (?cmd=s)
  fields:

    mega_id:
      description: ID меги, можно оставить пустым, тогда будут сохранены все зарегистрированные меги
      example: "mega"

get_port:
  description: Запросить текущий статус порта (или всех)
  fields:
    mega_id:
      description: ID меги, можно оставить пустым, тогда будут сохранены все зарегистрированные меги
      example: "mega"
    port:
      description: Номер порта (если не заполнять, будут запрошены все порты сразу)
      example: 1

run_cmd:
  description: Выполнить любую произвольную команду
  fields:

    mega_id:
      description: ID меги, можно оставить пустым, тогда будут сохранены все зарегистрированные меги
      example: "mega"
    port:
      description: Номер порта (это не порт, которым мы управляем, а порт с которого шлем команду)
      example: 1
    cmd:
      description: Любая поддерживаемая мегой команда
      example: "1:0"

```

## Состояния
Так же каждое устройство megad опрашивается на предмет работоспособности, текущий статус
хранится в mega.<id>

## Отладка
Если возникают проблемы, можно включить детальный лог, для этого в конфиг добавить:
```yaml
logger:
  default: info
  logs:
    custom_components.mega: debug
```