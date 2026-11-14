Вот полный готовый `README.md` — копируй целиком и вставляй в свой репозиторий на GitHub. Всё отрендерится: схема ASCII-артом, таблицы, код, заголовки.

```markdown
# 🔌 Интеграция ESPHome с SLS шлюзом через REST API v3

[![ESPHome](https://img.shields.io/badge/ESPHome-2026.9.1-blue)](https://esphome.io)
[![SLS](https://img.shields.io/badge/SLS-2026.08.30d5-green)](https://slsys.io)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2026.x-blueviolet)](https://home-assistant.io)
[![License](https://img.shields.io/badge/License-MIT-yellow)](LICENSE)

Полное руководство по управлению устройствами ESPHome через REST API v3 из Lua-скриптов SLS с интеграцией в Home Assistant через MQTT.

---

## 📑 Содержание

- [Введение](#введение)
- [Архитектура](#архитектура)
- [⚠️ Регистр букв — критически важно](#️-регистр-букв--критически-важно)
- [Особенности ESPHome REST API v3](#особенности-esphome-rest-api-v3)
- [Управление компонентами ESPHome](#управление-компонентами-esphome)
- [Интеграция с SLS](#интеграция-с-sls)
- [init.lua — инициализация](#initlua--инициализация)
- [Интеграция с Home Assistant](#интеграция-с-home-assistant)
- [Полный пример: LED-лента](#полный-пример-led-лента)
- [Цвета RGB](#цвета-rgb)
- [Отладка](#отладка)
- [FAQ](#faq)

---

## Введение

У тебя есть LED-лента на ESPHome, SLS шлюз для Zigbee и Home Assistant. Хочешь управлять лентой из HA, но не хочешь поднимать отдельный MQTT-брокер только для ESPHome?

**Решение:** SLS умеет работать с HTTP REST API ESPHome через Lua-скрипты и пробрасывать всё в HA через свой встроенный MQTT-клиент. Один шлюз — и Zigbee, и ESPHome, и HA в одном месте.

### Как это работает:

1. **ESPHome** прошивается с включённым `web_server` (REST API на порту 80)
2. **SLS** создаёт объекты в памяти и привязывает к ним Lua-скрипты
3. При изменении объекта скрипт шлёт HTTP-запрос на ESPHome
4. Флаг **M** (MQTT Notify) автоматически публикует состояние в MQTT
5. **Home Assistant** подписывается на топики и показывает устройство в UI

### Что получишь в итоге:

- ✅ Ползунок яркости в HA → лента меняет яркость
- ✅ Color picker в HA → лента меняет цвет
- ✅ Выбор эффекта в HA → лента запускает эффект
- ✅ Физический выключатель на ESPHome → HA видит изменение состояния
- ✅ Всё работает локально, без облаков и задержек

---

## Архитектура

```
┌────────────────────────────────────────────────────────────────────────────┐
│                           АРХИТЕКТУРА ИНТЕГРАЦИИ                           │
└────────────────────────────────────────────────────────────────────────────┘

  ┌──────────────────┐         ┌──────────────────┐         ┌──────────────────┐
  │                  │  MQTT   │                  │  HTTP   │                  │
  │  Home Assistant  │◄───────►│   MQTT Broker    │         │                  │
  │                  │  pub/   │   (Mosquitto)    │         │                  │
  │  • Lovelace UI   │  sub    │   :1883          │         │                  │
  │  • Автоматизации │         │                  │         │                  │
  └────────┬─────────┘         └────────┬─────────┘         │                  │
           │                            │                    │                  │
           │  [prefix]/obj/             │  [prefix]/obj/     │                  │
           │  detskaya_wled.*/set       │  detskaya_wled.*   │                  │
           │  (команды)                 │  (состояния)       │                  │
           │                            │                    │                  │
           │         ┌──────────────────┘                    │                  │
           │         │                                       │                  │
           ▼         ▼                                       │                  │
  ┌────────────────────────────────────────────┐             │                  │
  │                                            │             │                  │
  │              SLS Шлюз (ESP32)              │             │                  │
  │                                            │             │                  │
  │  ┌──────────────────────────────────────┐  │             │                  │
  │  │  Объекты (RAM):                      │  │             │                  │
  │  │  • detskaya_wled.brightness (INT)    │  │             │                  │
  │  │  • detskaya_wled.effect (STR)        │  │             │                  │
  │  │  • detskaya_wled.effect_speed (INT)  │  │             │                  │
  │  │  • detskaya_wled.color (STR)         │  │             │                  │
  │  └──────────────────────────────────────┘  │             │                  │
  │  ┌──────────────────────────────────────┐  │             │                  │
  │  │  Скрипты (Lua):                      │  │             │                  │
  │  │  • detskaya_wled.lua                 │  │             │                  │
  │  │  • detskaya_wled_effect.lua          │  │             │                  │
  │  │  • detskaya_wled_speed.lua           │  │             │                  │
  │  │  • detskaya_wled_color.lua           │  │             │                  │
  │  └──────────────────────────────────────┘  │             │                  │
  │  Флаги: M (MQTT), W (Write)                │             │                  │
  │  init.lua — автоинициализация              │             │                  │
  └─────────────────────┬──────────────────────┘             │                  │
                        │                                     │                  │
                        │  HTTP POST/GET                      │                  │
                        │  /light/strip/turn_on               │                  │
                        │  /number/effect_speed/set           │                  │
                        ▼                                     ▼                  │
              ┌──────────────────────────────────────────────────────┐          │
              │                                                      │          │
              │              ESPHome Device (ESP32-C3)               │          │
              │                                                      │          │
              │  • web_server v3 (:80)                               │          │
              │  • light: strip (WS2812, 287 LEDs)                   │          │
              │  • number: effect_speed                              │          │
              │  • effects: Matrix Rain, Fire, ...                   │          │
              │                                                      │          │
              └──────────────────────────────────────────────────────┘          │
                                                                                │
  ЛЕГЕНДА:                                                                      │
  ─────────────────────────────────────────────────────────────────────────     │
  ──►  MQTT (двусторонний, через брокер)                                        │
  ──►  HTTP (прямой, SLS → ESPHome)                                             │
  ──►  Внутренние процессы SLS (флаги M, W)                                     │
```

### Пошаговый поток команды:

1. **Пользователь** двигает ползунок яркости в HA
2. **HA** публикует в MQTT: `[prefix]/obj/detskaya_wled.brightness/set = 150`
3. **MQTT брокер** доставляет сообщение подписчикам
4. **SLS** автоматически подписан на `/set` (флаг M) — получает сообщение
5. **SLS** обновляет объект `detskaya_wled.brightness = 150`
6. **Флаг W** запускает скрипт `detskaya_wled.lua`
7. **Скрипт** шлёт HTTP POST: `/light/strip/turn_on?brightness=150`
8. **ESPHome** применяет яркость к ленте
9. **SLS** публикует состояние в MQTT (флаг M)
10. **HA** получает состояние и обновляет UI

---

## ⚠️ Регистр букв — критически важно

> 🚨 **Главная проблема, на которой ломается 90% интеграций!**
>
> MQTT чувствителен к регистру. `ZigbeeGate` и `ZIgbeeGate` — это **два разных топика**. То же самое с именами объектов: `detskaya_WLED` и `detskaya_wled` — разные объекты в SLS.

### Правила:

1. **Все имена объектов в SLS — строчными буквами** (`detskaya_wled.brightness`, не `detskaya_WLED.brightness`). HA генерирует топики в нижнем регистре.

2. **Системный префикс SLS — точно как в настройках**. Смотри в SLS → Settings → MQTT → "Системный префикс топика". Используй его везде.

3. **Имена компонентов ESPHome — только латиница в нижнем регистре** (`strip`, `effect_speed`, не `Strip`).

4. **Топики MQTT должны совпадать везде**: в init.lua, скриптах SLS, конфиге HA, MQTT Explorer.

### Правильная иерархия имён:

```
SLS объект:      detskaya_wled.brightness         (строчные)
MQTT топик:      [prefix]/obj/detskaya_wled.brightness
MQTT /set:       [prefix]/obj/detskaya_wled.brightness/set
HA command:      [prefix]/obj/detskaya_wled.brightness/set
HA state:        [prefix]/obj/detskaya_wled.brightness
ESPHome entity:  strip                            (латиница, строчные)
```

> ⚠️ Если есть дубликаты с разным регистром — удали их в SLS и приведи всё к единому регистру. Иначе будут "фантомные" объекты без скриптов.

---

## Особенности ESPHome REST API v3

> ⚠️ Эти особенности не очевидны из документации. Их игнорирование приводит к ошибкам 404, 411 и молчаливому игнорированию команд.

### 1. Content-Length обязателен для POST

```lua
-- Правильно:
local headers = "Content-Length: 2\r\n"
local body = "{}"

-- Неправильно (вернёт 411):
http.request(url, "POST")
```

### 2. Параметры в URL, а не в теле запроса

```lua
-- Правильно (brightness в URL):
local url = "http://<IP>/light/strip/turn_on?brightness=150"

-- Неправильно (проигнорируется):
local body = '{"brightness":150}'
```

### 3. Только латиница в именах компонентов

```yaml
# Правильно:
light:
  - platform: esp32_rmt_led_strip
    name: "strip"

# Неправильно (404):
light:
  - platform: esp32_rmt_led_strip
    name: "Лента"
```

### 4. Пустое тело запроса

Для POST используется пустой JSON `{}` с `Content-Length: 2`.

### 5. Обязательный color_mode для RGB

```lua
local url = "http://<IP>/light/strip/turn_on?color_mode=rgb&r=255&g=0&b=0"
```

Без `color_mode=rgb` ESPHome может игнорировать R, G, B (баг #5750).

---

## Управление компонентами ESPHome

### Light

```yaml
light:
  - platform: esp32_rmt_led_strip
    name: "strip"
    id: strip
    pin: GPIO4
    num_leds: 287
    chipset: WS2812
    channel_colors: RGB
```

```lua
-- Включение с яркостью:
local url = "http://<IP>/light/strip/turn_on?brightness=150"
http.request(url, "POST", "Content-Length: 2\r\n", "{}")

-- Выключение:
local url = "http://<IP>/light/strip/turn_off"
http.request(url, "POST", "Content-Length: 2\r\n", "{}")

-- Эффект:
local effect = "Matrix%20Rain"
local url = "http://<IP>/light/strip/turn_on?effect=" .. effect
http.request(url, "POST", "Content-Length: 2\r\n", "{}")

-- Цвет RGB:
local url = "http://<IP>/light/strip/turn_on?color_mode=rgb&r=255&g=0&b=0"
http.request(url, "POST", "Content-Length: 2\r\n", "{}")
```

### Number

```yaml
number:
  - platform: template
    name: "effect_speed"
    id: effect_speed
    min_value: 1
    max_value: 100
    step: 1
    initial_value: 20
    mode: slider
```

```lua
local url = "http://<IP>/number/effect_speed/set?value=50"
http.request(url, "POST", "Content-Length: 2\r\n", "{}")
```

### Switch

```yaml
switch:
  - platform: gpio
    name: "relay"
    pin: GPIO5
```

```lua
local url = "http://<IP>/switch/relay/turn_on"
http.request(url, "POST", "Content-Length: 2\r\n", "{}")
```

### Другие компоненты

| Компонент | Действие | URL |
|-----------|----------|-----|
| Fan | Включить | `/fan/ceiling_fan/turn_on?speed_level=2` |
| Cover | Открыть | `/cover/blinds/open` |
| Cover | Позиция | `/cover/blinds/set?position=0.5` |
| Select | Опция | `/select/mode/set?option=auto` |
| Button | Нажать | `/button/reboot/press` |
| Sensor | Читать (GET) | `/sensor/temperature` |

---

## Интеграция с SLS

> 💡 В SLS объекты хранятся **только в памяти** и не сохраняются во flash. Поэтому инициализация делается в `init.lua`.

### Основные функции:

```lua
obj.setType("detskaya_wled.brightness", "INT")   -- тип
obj.set("detskaya_wled.brightness", 150)           -- значение
local val = obj.get("detskaya_wled.brightness")    -- получить (может быть nil!)
obj.setScript("detskaya_wled.brightness", "detskaya_wled.lua", true)  -- скрипт + флаг W
obj.setShare("detskaya_wled.brightness", true, false)  -- флаг M (MQTT)
obj.remove("detskaya_wled.brightness")             -- удалить
```

### Флаги объектов:

| Флаг | Название | Описание |
|------|----------|----------|
| **M** | MQTT Notify | Публикация в MQTT + автоподписка на `/set` |
| **W** | Write | Скрипт вызывается при каждой записи |
| **A** | ACK | Обратная связь (для предотвращения циклов) |
| **N** | Network | Синхронизация между шлюзами SLS |

> 🚨 В SLS значение объекта передаётся через `obj.get("имя")`, а НЕ через глобальную переменную `value`. Последний вернёт `nil`!

```lua
-- Правильно:
local brightness = obj.get("detskaya_wled.brightness")
local num = tonumber(brightness) or 0

-- Неправильно (вернёт nil):
local brightness = value
```

---

## init.lua — инициализация

> 🚨 Все имена объектов — **строчными буквами**! HA публикует в нижнем регистре.

### Как работает флаг M:

- `obj.setShare(name, true, false)` устанавливает флаг M
- SLS **автоматически публикует** изменения в топик `.../obj/имя`
- SLS **автоматически подписывается** на топик `.../obj/имя/set`
- При получении сообщения в `/set` — SLS сам обновляет объект
- **Никакого явного `mqtt.sub()` не нужно!**

### init.lua:

```lua
-- init.lua — Инициализация SLS шлюза

-- 1. Zigbee скрипты
obj.setType("isDay", "BOOL")
scripts.run("day_night.lua")
scripts.setTimer("day_night.lua", 60)

-- 2. Конфигурация ESPHome объектов (ВСЕ МАЛЕНЬКИМИ БУКВАМИ!)
local devices = {
    {name = "detskaya_wled.brightness",    type = "INT",  default = 255,  script = "detskaya_wled.lua"},
    {name = "detskaya_wled.effect",         type = "STR",  default = "None", script = "detskaya_wled_effect.lua"},
    {name = "detskaya_wled.effect_speed",   type = "INT",  default = 20,   script = "detskaya_wled_speed.lua"},
    {name = "detskaya_wled.color",          type = "STR",  default = "FFFFFF", script = "detskaya_wled_color.lua"}
}

-- 3. Инициализация и настройка
for _, dev in ipairs(devices) do
    local current = obj.get(dev.name)

    if current == nil then
        obj.setType(dev.name, dev.type)
        obj.set(dev.name, dev.default)
    end

    -- ПРИНУДИТЕЛЬНО устанавливаем скрипт и флаги при каждой загрузке
    obj.setScript(dev.name, dev.script, true)   -- флаг W
    obj.setShare(dev.name, true, false)          -- флаг M
end
```

### Что делает скрипт:

- ✅ Создаёт объекты с правильными типами и дефолтами
- ✅ Привязывает скрипты (флаг W) при каждой загрузке
- ✅ Включает MQTT (флаг M) — SLS сам подписывается на `/set`
- ✅ HA → MQTT → SLS → ESPHome работает без мостов

---

## Интеграция с Home Assistant

### Требования:

- MQTT брокер (Mosquitto) работает
- SLS подключён к MQTT (Settings → MQTT → Enable)
- HA подключён к тому же MQTT брокеру
- У объектов SLS установлен флаг M

### Через YAML в configuration.yaml (рекомендуется):

```yaml
mqtt:
  light:
    - name: "Подсветка детская"
      unique_id: "wled_podsvetka_detskaya"

      command_topic: "[prefix]/obj/detskaya_wled.brightness/set"
      state_topic: "[prefix]/obj/detskaya_wled.brightness"

      brightness_command_topic: "[prefix]/obj/detskaya_wled.brightness/set"
      brightness_state_topic: "[prefix]/obj/detskaya_wled.brightness"
      brightness_scale: 255

      rgb_command_topic: "[prefix]/obj/detskaya_wled.color/set"
      rgb_state_topic: "[prefix]/obj/detskaya_wled.color"
      rgb_command_template: "{{ '%02X%02X%02X' | format(red, green, blue) }}"
      rgb_value_template: "{{ value[0:2]|int(base=16) }},{{ value[2:4]|int(base=16) }},{{ value[4:6]|int(base=16) }}"

      effect_command_topic: "[prefix]/obj/detskaya_wled.effect/set"
      effect_state_topic: "[prefix]/obj/detskaya_wled.effect"
      effect_list:
        - "None"
        - "Random"
        - "Strobe"
        - "Flicker"
        - "Pulse"
        - "Rainbow"
        - "Matrix Rain"
        - "Fire"

      state_value_template: "{% if value | int > 0 %}ON{% else %}OFF{% endif %}"
      payload_on: "255"
      payload_off: "0"
      on_command_type: "brightness"  # ВАЖНО!

      optimistic: true
      qos: 0
      schema: "basic"
```

> 🚨 **Ключевые параметры:**
>
> - `on_command_type: brightness` — без этого HA шлёт только 0/255
> - `optimistic: true` — HA не ждёт подтверждения
> - `payload_on: "255"` и `payload_off: "0"` — числа, не ON/OFF
> - **Регистр во всех топиках** должен совпадать с SLS!

### Через веб-интерфейс HA:

1. Settings → Devices & Services → MQTT → Configure → Add Entity → Light
2. **Name:** `Подсветка детская`
3. **Unique ID:** `wled_podsvetka_detskaya`
4. **Schema:** `Default schema`
5. **Command topic:** `[prefix]/obj/detskaya_wled.brightness/set`
6. **State topic:** `[prefix]/obj/detskaya_wled.brightness`
7. **Тип команды ВКЛ:** `Яркость` (Brightness) — **ОБЯЗАТЕЛЬНО!**

#### Brightness settings:
- Brightness command topic: `[prefix]/obj/detskaya_wled.brightness/set`
- Brightness state topic: `[prefix]/obj/detskaya_wled.brightness`
- Brightness scale: `255`

#### RGB color settings:
- RGB command topic: `[prefix]/obj/detskaya_wled.color/set`
- RGB state topic: `[prefix]/obj/detskaya_wled.color`
- RGB command template: `{{ '%02X%02X%02X' | format(red, green, blue) }}`
- RGB value template: `{{ value[0:2]|int(base=16) }},{{ value[2:4]|int(base=16) }},{{ value[4:6]|int(base=16) }}`

#### Effect settings:
- Effect command topic: `[prefix]/obj/detskaya_wled.effect/set`
- Effect state topic: `[prefix]/obj/detskaya_wled.effect`
- Список эффектов: None, Random, Strobe, Flicker, Pulse, Rainbow, Matrix Rain, Fire

#### Other settings:
- State value template: `{% if value | int > 0 %}ON{% else %}OFF{% endif %}`
- Payload on: `255`
- Payload off: `0`
- Optimistic: ✅
- Retain: ❌

---

## Полный пример: LED-лента

### ESPHome YAML:

```yaml
esphome:
  name: wled-c3

esp32:
  variant: esp32c3
  framework:
    type: esp-idf

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

web_server:
  port: 80
  version: 3

light:
  - platform: esp32_rmt_led_strip
    name: "strip"
    id: strip
    pin: GPIO4
    num_leds: 287
    chipset: WS2812
    channel_colors: RGB

number:
  - platform: template
    name: "effect_speed"
    id: effect_speed
    min_value: 1
    max_value: 100
    step: 1
    initial_value: 20
    mode: slider
```

### Скрипты SLS:

**detskaya_wled.lua** (яркость):

```lua
local ip_esp = "<IP_ESP>"
local entity = "strip"

local raw_val = obj.get("detskaya_wled.brightness")
local brightness = tonumber(raw_val) or 0
if brightness < 0 then brightness = 0 end

local headers = "Content-Length: 2\r\n"
local body = "{}"

if brightness > 0 then
    local url = "http://" .. ip_esp .. "/light/" .. entity .. "/turn_on?brightness=" .. brightness
    http.request(url, "POST", headers, body)
else
    local url = "http://" .. ip_esp .. "/light/" .. entity .. "/turn_off"
    http.request(url, "POST", headers, body)
end
```

**detskaya_wled_effect.lua** (эффект):

```lua
local ip_esp = "<IP_ESP>"
local entity = "strip"

local raw_effect = obj.get("detskaya_wled.effect")
if raw_effect == nil or raw_effect == "" then return end

local effect_name = string.gsub(raw_effect, " ", "%%20")
local headers = "Content-Length: 2\r\n"
local body = "{}"
local url = "http://" .. ip_esp .. "/light/" .. entity .. "/turn_on?effect=" .. effect_name
http.request(url, "POST", headers, body)
```

**detskaya_wled_speed.lua** (скорость):

```lua
local ip_esp = "<IP_ESP>"
local raw_speed = obj.get("detskaya_wled.effect_speed")
local speed = tonumber(raw_speed) or 20
if speed < 1 then speed = 1 end
if speed > 100 then speed = 100 end

local headers = "Content-Length: 2\r\n"
local body = "{}"
local url = "http://" .. ip_esp .. "/number/effect_speed/set?value=" .. speed
http.request(url, "POST", headers, body)
```

**detskaya_wled_color.lua** (цвет):

```lua
local ip_esp = "<IP_ESP>"
local entity = "strip"

local hex = obj.get("detskaya_wled.color")
if hex == nil or hex == "" then return end

hex = string.upper(hex)
hex = string.gsub(hex, "#", "")
if string.len(hex) ~= 6 then return end

local r = tonumber(string.sub(hex, 1, 2), 16) or 0
local g = tonumber(string.sub(hex, 3, 4), 16) or 0
local b = tonumber(string.sub(hex, 5, 6), 16) or 0

local headers = "Content-Length: 2\r\n"
local body = "{}"
local url = "http://" .. ip_esp .. "/light/" .. entity .. "/turn_on?color_mode=rgb&r=" .. r .. "&g=" .. g .. "&b=" .. b
http.request(url, "POST", headers, body)
```

### Таблица объектов SLS:

| Имя объекта | Тип | Default | Скрипт | Флаги |
|-------------|-----|---------|--------|-------|
| `detskaya_wled.brightness` | INT | 255 | detskaya_wled.lua | MW |
| `detskaya_wled.effect` | STR | None | detskaya_wled_effect.lua | MW |
| `detskaya_wled.effect_speed` | INT | 20 | detskaya_wled_speed.lua | MW |
| `detskaya_wled.color` | STR | FFFFFF | detskaya_wled_color.lua | MW |

---

## Цвета RGB

Вместо трёх объектов для R, G, B — один с HEX-кодом.

| Цвет | HEX |
|------|-----|
| ⚪ Белый | `FFFFFF` |
| ⚫ Чёрный | `000000` |
| 🔴 Красный | `FF0000` |
| 🟢 Зелёный | `00FF00` |
| 🔵 Синий | `0000FF` |
| 🟡 Жёлтый | `FFFF00` |
| 🩵 Голубой | `00FFFF` |
| 🟣 Пурпурный | `FF00FF` |
| 🟠 Оранжевый | `FFA500` |
| 🌸 Розовый | `FFC0CB` |
| 🟪 Фиолетовый | `800080` |
| 🕯️ Тёплый белый | `FFE4B5` |
| ❄️ Холодный белый | `F0F8FF` |

> 💡 Можно вводить с `#` или без — скрипт поймёт оба варианта.

---

## Отладка

### Проверка ESPHome:

```bash
curl http://<IP_ESP>/light/strip
```

Должен вернуть JSON:
```json
{
  "id": "light/strip",
  "state": "OFF",
  "brightness": 255,
  "color": {"r": 255, "g": 255, "b": 255}
}
```

### Проверка через curl:

```bash
# Включение с яркостью
curl -X POST -H "Content-Length: 2" -d "{}" "http://<IP>/light/strip/turn_on?brightness=150"

# Выключение
curl -X POST -H "Content-Length: 2" -d "{}" "http://<IP>/light/strip/turn_off"

# Цвет
curl -X POST -H "Content-Length: 2" -d "{}" "http://<IP>/light/strip/turn_on?color_mode=rgb&r=255&g=0&b=0"
```

### Проверка MQTT:

```bash
# Подписка на все топики SLS
mosquitto_sub -h <MQTT_BROKER_IP> -t "[prefix]/obj/#" -v

# Публикация команды
mosquitto_pub -h <MQTT_BROKER_IP> -t "[prefix]/obj/detskaya_wled.brightness/set" -m "150"
```

### Таблица ошибок:

| Ошибка | Причина | Решение |
|--------|---------|---------|
| `404 Not Found` | Неправильное имя или кириллица | Только латиница в YAML |
| `411 Length Required` | Нет Content-Length | Добавь `Content-Length: 2` |
| `Error -1` в SLS | HTTP не дошёл | Проверь IP и доступность |
| HA не управляет | Неправильный регистр | Строчные буквы везде |
| HA шлёт только 0/255 | Нет `on_command_type` | Установи `on_command_type: brightness` |
| Дубликаты объектов | Разный регистр | Удали дубликаты |
| TemplateSyntaxError | Многострочный шаблон | Используй YAML |
| ValueError: invalid literal | rgb_value_template = список | Убери `[]` в шаблоне |

---

## FAQ

**Q: Почему нельзя кириллицу в именах ESPHome?**
A: ESPHome REST API не поддерживает URL-encoding кириллицы.

**Q: Почему POST без Content-Length возвращает 411?**
A: Требование ESP-IDF HTTP-сервера.

**Q: Почему JSON-тело игнорируется?**
A: ESPHome web_server v3 читает параметры только из URL query-строки.

**Q: Почему value в Lua-скрипте всегда nil?**
A: В SLS значение через `obj.get("имя")`, не через глобальную `value`.

**Q: Почему объекты сбрасываются после перезагрузки SLS?**
A: Объекты в RAM. Используй init.lua с принудительной установкой скриптов и флагов.

**Q: Почему HA шлёт только 0/255?**
A: Установи `on_command_type: brightness`.

**Q: Почему дубликаты объектов в SLS?**
A: Разный регистр. HA генерирует топики в нижнем регистре — используй строчные буквы везде.

**Q: Почему команды из HA не доходят до SLS?**
A: Проверь регистр. SLS автоматически подписывается на `/set` благодаря флагу M.

**Q: Как проверить MQTT?**
A: `mosquitto_sub -t "[prefix]/obj/#" -v`. Меняй значение в SLS — должно появиться сообщение.

**Q: Можно управлять несколькими устройствами?**
A: Да. Отдельные объекты с разными префиксами (`kitchen_wled.*`, `bedroom_wled.*`) и разные IP.

**Q: Работает без интернета?**
A: Да, всё локально.

**Q: Нужен ли MQTT для базовой работы?**
A: Нет. SLS управляет ESPHome напрямую через HTTP. MQTT только для HA.

**Q: Нужен ли флаг ACK?**
A: В базовой схеме (HA → SLS → ESPHome) — нет. Нужен только при обратной синхронизации (polling ESPHome → SLS).

---

## 📝 Лицензия

MIT License. Свободно используй, модифицируй, распространяй.

## 👤 Автор

**@diemon24** — IT-шник всю жизнь, руковожу IT проектами.

## 🔧 Версии тестирования

| Компонент | Версия |
|-----------|--------|
| ESPHome | 2026.9.1 |
| SLS | 2026.08.30d5 |
| Home Assistant | 2026.x |
| Mosquitto | 2.x |

## 💡 Ключевые инсайты

- **Регистр букв** — главная причина 90% багов
- **Флаг M** автоматически подписывается на `/set` — не нужен явный `mqtt.sub()`
- **`on_command_type: brightness`** — без этого HA шлёт только 0/255
- **init.lua** должен принудительно устанавливать скрипты и флаги при каждой загрузке
- **Content-Length: 2** — обязательно для всех POST запросов к ESPHome
```

## Как использовать:

1. Создай файл `README.md` в корне своего репозитория на GitHub
2. Скопируй весь код выше и вставь в файл
3. Замени `[prefix]` на свой системный префикс SLS (например, `ZigbeeGate_dacha_2floor`)
4. Замени `<IP_ESP>` и `<MQTT_BROKER_IP>` на реальные IP
5. Закоммить и запушь:

```bash
git add README.md
git commit -m "docs: интеграция ESPHome с SLS"
git push
```

GitHub автоматически отрендерит Markdown — схема ASCII-артом отобразится как моноширинный текст, таблицы и код будут с подсветкой.
