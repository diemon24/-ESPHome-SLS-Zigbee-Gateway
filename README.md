<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Интеграция ESPHome с SLS шлюзом через REST API v3</title>
<style>
  :root {
    --bg: #0f1419;
    --surface: #1a1f2e;
    --surface-2: #242b3d;
    --border: #2d3548;
    --text: #e6edf3;
    --text-muted: #8b949e;
    --accent: #58a6ff;
    --accent-2: #3fb950;
    --accent-3: #f78166;
    --code-bg: #0d1117;
    --warning: #d29922;
  }
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    background: var(--bg);
    color: var(--text);
    line-height: 1.6;
    padding: 0;
  }
  header {
    background: linear-gradient(135deg, #1a1f2e 0%, #242b3d 100%);
    padding: 40px 20px;
    border-bottom: 2px solid var(--accent);
  }
  header h1 { font-size: 2.2em; color: var(--accent); margin-bottom: 10px; }
  header p { color: var(--text-muted); font-size: 1.1em; }
  nav {
    background: var(--surface);
    padding: 15px 20px;
    position: sticky;
    top: 0;
    z-index: 100;
    border-bottom: 1px solid var(--border);
  }
  nav a {
    color: var(--accent);
    text-decoration: none;
    margin-right: 20px;
    font-weight: 500;
    font-size: 0.95em;
  }
  nav a:hover { color: var(--accent-2); }
  main { max-width: 1100px; margin: 0 auto; padding: 40px 20px; }
  section {
    background: var(--surface);
    padding: 30px;
    margin-bottom: 30px;
    border-radius: 8px;
    border: 1px solid var(--border);
  }
  h2 {
    color: var(--accent);
    font-size: 1.8em;
    margin-bottom: 20px;
    padding-bottom: 10px;
    border-bottom: 1px solid var(--border);
  }
  h3 { color: var(--accent-2); font-size: 1.3em; margin: 25px 0 15px 0; }
  h4 { color: var(--accent-3); margin: 15px 0 10px 0; }
  p, li { margin-bottom: 10px; }
  ul, ol { margin-left: 25px; margin-bottom: 15px; }
  code {
    background: var(--code-bg);
    padding: 2px 6px;
    border-radius: 4px;
    font-family: 'JetBrains Mono', 'Consolas', monospace;
    font-size: 0.9em;
    color: var(--accent-3);
  }
  pre {
    background: var(--code-bg);
    padding: 20px;
    border-radius: 6px;
    overflow-x: auto;
    margin: 15px 0;
    border-left: 3px solid var(--accent);
  }
  pre code {
    background: none;
    padding: 0;
    color: var(--text);
    font-size: 0.9em;
    line-height: 1.5;
  }
  .comment { color: #8b949e; font-style: italic; }
  .keyword { color: #ff7b72; }
  .string { color: #a5d6ff; }
  .number { color: #79c0ff; }
  table {
    width: 100%;
    border-collapse: collapse;
    margin: 15px 0;
    background: var(--surface-2);
  }
  th, td {
    padding: 12px;
    text-align: left;
    border-bottom: 1px solid var(--border);
  }
  th { background: var(--surface); color: var(--accent); font-weight: 600; }
  .warning {
    background: rgba(210, 153, 34, 0.1);
    border-left: 4px solid var(--warning);
    padding: 15px;
    margin: 15px 0;
    border-radius: 4px;
  }
  .warning strong { color: var(--warning); }
  .success {
    background: rgba(63, 185, 80, 0.1);
    border-left: 4px solid var(--accent-2);
    padding: 15px;
    margin: 15px 0;
    border-radius: 4px;
  }
  .info {
    background: rgba(88, 166, 255, 0.1);
    border-left: 4px solid var(--accent);
    padding: 15px;
    margin: 15px 0;
    border-radius: 4px;
  }
  .critical {
    background: rgba(247, 129, 102, 0.15);
    border-left: 4px solid #f78166;
    padding: 15px;
    margin: 15px 0;
    border-radius: 4px;
  }
  .critical strong { color: #f78166; }
  .architecture-diagram {
    background: var(--surface-2);
    padding: 20px;
    border-radius: 8px;
    margin: 20px 0;
    text-align: center;
  }
  footer {
    text-align: center;
    padding: 30px;
    color: var(--text-muted);
    border-top: 1px solid var(--border);
    margin-top: 40px;
  }
  @media print {
    body { background: white; color: black; }
    section { background: white; border: 1px solid #ccc; }
    pre { background: #f5f5f5; }
    nav { display: none; }
  }
</style>
</head>
<body>

<header>
  <h1>🔌 Интеграция ESPHome с SLS шлюзом</h1>
  <p>Полное руководство по управлению устройствами ESPHome через REST API v3 из Lua-скриптов SLS с интеграцией в Home Assistant через MQTT</p>
</header>

<nav>
  <a href="#intro">Введение</a>
  <a href="#architecture">🏗️ Архитектура</a>
  <a href="#critical">⚠️ Регистр</a>
  <a href="#features">Особенности API</a>
  <a href="#components">Компоненты</a>
  <a href="#sls">Интеграция с SLS</a>
  <a href="#init">init.lua</a>
  <a href="#ha">Интеграция с HA</a>
  <a href="#example">Полный пример</a>
  <a href="#colors">Цвета RGB</a>
  <a href="#debug">Отладка</a>
  <a href="#faq">FAQ</a>
</nav>

<main>

<section id="intro">
  <h2>Введение</h2>
  <p><strong>ESPHome</strong> — фреймворк для прошивки ESP32/ESP8266 устройств. Начиная с версии 2026.x, ESPHome включает встроенный веб-сервер с REST API для управления компонентами без Home Assistant или MQTT.</p>
  <p>Эта инструкция описывает, как интегрировать ESPHome устройства с <strong>SLS шлюзом</strong> через Lua-скрипты, используя HTTP REST API, и пробросить управление ими в <strong>Home Assistant</strong> через MQTT.</p>
  
  <div class="info">
    <strong>Преимущества подхода:</strong>
    <ul>
      <li>Задержка менее 100 мс</li>
      <li>Совместимо с любой версией SLS, поддерживающей Lua и HTTP</li>
      <li>Двусторонняя синхронизация между HA и ESPHome через SLS</li>
    </ul>
  </div>
</section>

<section id="architecture">
  <h2>🏗️ Архитектура интеграции</h2>
  
  <div class="architecture-diagram">
    <svg width="900" height="600" xmlns="http://www.w3.org/2000/svg">
      <defs>
        <!-- Градиенты -->
        <linearGradient id="grad-esphome" x1="0%" y1="0%" x2="100%" y2="100%">
          <stop offset="0%" style="stop-color:#3fb950;stop-opacity:1" />
          <stop offset="100%" style="stop-color:#238636;stop-opacity:1" />
        </linearGradient>
        <linearGradient id="grad-sls" x1="0%" y1="0%" x2="100%" y2="100%">
          <stop offset="0%" style="stop-color:#58a6ff;stop-opacity:1" />
          <stop offset="100%" style="stop-color:#1f6feb;stop-opacity:1" />
        </linearGradient>
        <linearGradient id="grad-mqtt" x1="0%" y1="0%" x2="100%" y2="100%">
          <stop offset="0%" style="stop-color:#f78166;stop-opacity:1" />
          <stop offset="100%" style="stop-color:#da3633;stop-opacity:1" />
        </linearGradient>
        <linearGradient id="grad-ha" x1="0%" y1="0%" x2="100%" y2="100%">
          <stop offset="0%" style="stop-color:#a371f7;stop-opacity:1" />
          <stop offset="100%" style="stop-color:#7c3aed;stop-opacity:1" />
        </linearGradient>
        
        <!-- Стрелки -->
        <marker id="arrow-blue" markerWidth="10" markerHeight="10" refX="9" refY="3" orient="auto" markerUnits="strokeWidth">
          <path d="M0,0 L0,6 L9,3 z" fill="#58a6ff" />
        </marker>
        <marker id="arrow-green" markerWidth="10" markerHeight="10" refX="9" refY="3" orient="auto" markerUnits="strokeWidth">
          <path d="M0,0 L0,6 L9,3 z" fill="#3fb950" />
        </marker>
        <marker id="arrow-orange" markerWidth="10" markerHeight="10" refX="9" refY="3" orient="auto" markerUnits="strokeWidth">
          <path d="M0,0 L0,6 L9,3 z" fill="#f78166" />
        </marker>
        <marker id="arrow-purple" markerWidth="10" markerHeight="10" refX="9" refY="3" orient="auto" markerUnits="strokeWidth">
          <path d="M0,0 L0,6 L9,3 z" fill="#a371f7" />
        </marker>
      </defs>
      
      <!-- Фон -->
      <rect width="900" height="600" fill="#0f1419" rx="10"/>
      
      <!-- ESPHome устройство (слева) -->
      <rect x="50" y="200" width="180" height="200" fill="url(#grad-esphome)" rx="10" opacity="0.9"/>
      <text x="140" y="240" font-family="Arial" font-size="18" font-weight="bold" fill="white" text-anchor="middle">ESPHome</text>
      <text x="140" y="265" font-family="Arial" font-size="14" fill="white" text-anchor="middle">WLED устройство</text>
      <text x="140" y="290" font-family="Arial" font-size="12" fill="white" text-anchor="middle">REST API v3</text>
      <text x="140" y="315" font-family="Arial" font-size="11" fill="white" text-anchor="middle">HTTP :80</text>
      <text x="140" y="345" font-family="Arial" font-size="11" fill="white" text-anchor="middle">/light/strip</text>
      <text x="140" y="365" font-family="Arial" font-size="11" fill="white" text-anchor="middle">/number/effect_speed</text>
      
      <!-- SLS шлюз (центр) -->
      <rect x="350" y="150" width="200" height="300" fill="url(#grad-sls)" rx="10" opacity="0.9"/>
      <text x="450" y="190" font-family="Arial" font-size="20" font-weight="bold" fill="white" text-anchor="middle">SLS Шлюз</text>
      <text x="450" y="220" font-family="Arial" font-size="14" fill="white" text-anchor="middle">ESP32 + Lua</text>
      <text x="450" y="250" font-family="Arial" font-size="12" fill="white" text-anchor="middle">Объекты:</text>
      <text x="450" y="270" font-family="monospace" font-size="11" fill="white" text-anchor="middle">detskaya_wled.*</text>
      <text x="450" y="295" font-family="Arial" font-size="12" fill="white" text-anchor="middle">Скрипты:</text>
      <text x="450" y="315" font-family="monospace" font-size="10" fill="white" text-anchor="middle">detskaya_wled.lua</text>
      <text x="450" y="335" font-family="monospace" font-size="10" fill="white" text-anchor="middle">detskaya_wled_color.lua</text>
      <text x="450" y="360" font-family="Arial" font-size="12" fill="white" text-anchor="middle">Флаги: M, W</text>
      <text x="450" y="385" font-family="Arial" font-size="11" fill="white" text-anchor="middle">init.lua</text>
      <text x="450" y="410" font-family="Arial" font-size="11" fill="white" text-anchor="middle">MQTT клиент</text>
      
      <!-- MQTT брокер (справа вверху) -->
      <rect x="650" y="50" width="200" height="150" fill="url(#grad-mqtt)" rx="10" opacity="0.9"/>
      <text x="750" y="90" font-family="Arial" font-size="18" font-weight="bold" fill="white" text-anchor="middle">MQTT Брокер</text>
      <text x="750" y="115" font-family="Arial" font-size="13" fill="white" text-anchor="middle">Mosquitto</text>
      <text x="750" y="140" font-family="Arial" font-size="11" fill="white" text-anchor="middle">:1883</text>
      <text x="750" y="165" font-family="monospace" font-size="10" fill="white" text-anchor="middle">[prefix]/obj/</text>
      <text x="750" y="180" font-family="monospace" font-size="10" fill="white" text-anchor="middle">detskaya_wled.*</text>
      
      <!-- Home Assistant (справа внизу) -->
      <rect x="650" y="350" width="200" height="200" fill="url(#grad-ha)" rx="10" opacity="0.9"/>
      <text x="750" y="390" font-family="Arial" font-size="18" font-weight="bold" fill="white" text-anchor="middle">Home Assistant</text>
      <text x="750" y="420" font-family="Arial" font-size="13" fill="white" text-anchor="middle">UI + Автоматизации</text>
      <text x="750" y="450" font-family="Arial" font-size="12" fill="white" text-anchor="middle">MQTT интеграция</text>
      <text x="750" y="480" font-family="Arial" font-size="11" fill="white" text-anchor="middle">configuration.yaml</text>
      <text x="750" y="510" font-family="Arial" font-size="11" fill="white" text-anchor="middle">Lovelace UI</text>
      <text x="750" y="535" font-family="Arial" font-size="11" fill="white" text-anchor="middle">Color Picker</text>
      
      <!-- Стрелки и подписи -->
      
      <!-- ESPHome ↔ SLS (HTTP) -->
      <line x1="230" y1="300" x2="350" y2="300" stroke="#3fb950" stroke-width="3" marker-end="url(#arrow-green)"/>
      <text x="290" y="285" font-family="Arial" font-size="11" fill="#3fb950" text-anchor="middle">HTTP POST</text>
      <text x="290" y="320" font-family="Arial" font-size="10" fill="#3fb950" text-anchor="middle">/light/strip/turn_on</text>
      
      <line x1="350" y1="330" x2="230" y2="330" stroke="#3fb950" stroke-width="3" marker-end="url(#arrow-green)"/>
      <text x="290" y="350" font-family="Arial" font-size="11" fill="#3fb950" text-anchor="middle">HTTP GET</text>
      <text x="290" y="365" font-family="Arial" font-size="10" fill="#3fb950" text-anchor="middle">/light/strip</text>
      
      <!-- SLS → MQTT (публикация) -->
      <line x1="550" y1="220" x2="650" y2="125" stroke="#f78166" stroke-width="3" marker-end="url(#arrow-orange)"/>
      <text x="600" y="160" font-family="Arial" font-size="11" fill="#f78166" text-anchor="middle">MQTT Publish</text>
      <text x="600" y="175" font-family="Arial" font-size="10" fill="#f78166" text-anchor="middle">Флаг M</text>
      
      <!-- MQTT → SLS (подписка) -->
      <line x1="650" y1="155" x2="550" y2="250" stroke="#f78166" stroke-width="3" marker-end="url(#arrow-orange)"/>
      <text x="600" y="195" font-family="Arial" font-size="11" fill="#f78166" text-anchor="middle">MQTT Subscribe</text>
      <text x="600" y="210" font-family="Arial" font-size="10" fill="#f78166" text-anchor="middle">/set топики</text>
      
      <!-- MQTT ↔ HA -->
      <line x1="750" y1="200" x2="750" y2="350" stroke="#a371f7" stroke-width="3" marker-end="url(#arrow-purple)"/>
      <line x1="750" y1="350" x2="750" y2="200" stroke="#a371f7" stroke-width="3" marker-end="url(#arrow-purple)"/>
      <text x="770" y="275" font-family="Arial" font-size="11" fill="#a371f7">MQTT</text>
      <text x="770" y="290" font-family="Arial" font-size="10" fill="#a371f7">двусторонняя</text>
      
      <!-- Легенда -->
      <rect x="50" y="50" width="250" height="120" fill="#1a1f2e" rx="5" opacity="0.95"/>
      <text x="60" y="75" font-family="Arial" font-size="14" font-weight="bold" fill="white">Легенда:</text>
      <line x1="60" y1="95" x2="100" y2="95" stroke="#3fb950" stroke-width="3"/>
      <text x="110" y="100" font-family="Arial" font-size="11" fill="white">HTTP (ESPHome ↔ SLS)</text>
      <line x1="60" y1="115" x2="100" y2="115" stroke="#f78166" stroke-width="3"/>
      <text x="110" y="120" font-family="Arial" font-size="11" fill="white">MQTT (SLS ↔ Брокер)</text>
      <line x1="60" y1="135" x2="100" y2="135" stroke="#a371f7" stroke-width="3"/>
      <text x="110" y="140" font-family="Arial" font-size="11" fill="white">MQTT (Брокер ↔ HA)</text>
      <line x1="60" y1="155" x2="100" y2="155" stroke="#58a6ff" stroke-width="3"/>
      <text x="110" y="160" font-family="Arial" font-size="11" fill="white">Внутренние процессы SLS</text>
    </svg>
  </div>

  <h3>Как это работает (пошагово):</h3>
  
  <ol>
    <li><strong>Пользователь</strong> двигает ползунок яркости в Home Assistant UI</li>
    <li><strong>HA</strong> публикует значение в MQTT топик: <code>[prefix]/obj/detskaya_wled.brightness/set = 150</code></li>
    <li><strong>MQTT брокер</strong> доставляет сообщение всем подписчикам</li>
    <li><strong>SLS</strong> автоматически подписан на топик <code>/set</code> (благодаря флагу M) и получает сообщение</li>
    <li><strong>SLS</strong> обновляет объект <code>detskaya_wled.brightness = 150</code></li>
    <li><strong>Флаг W</strong> запускает скрипт <code>detskaya_wled.lua</code></li>
    <li><strong>Скрипт</strong> делает HTTP POST на ESPHome: <code>/light/strip/turn_on?brightness=150</code></li>
    <li><strong>ESPHome</strong> применяет яркость к LED-ленте</li>
    <li><strong>SLS</strong> публикует новое состояние в MQTT: <code>[prefix]/obj/detskaya_wled.brightness = 150</code> (флаг M)</li>
    <li><strong>HA</strong> получает состояние и обновляет UI</li>
  </ol>

  <div class="success">
    <strong>✅ Ключевые моменты:</strong>
    <ul>
      <li>SLS автоматически подписывается на топики <code>/set</code> благодаря флагу M — никакого явного <code>mqtt.sub()</code> не нужно</li>
      <li>Все имена объектов — строчными буквами для совместимости с HA</li>
      <li>init.lua принудительно устанавливает скрипты и флаги при каждой загрузке</li>
      <li>Полный цикл: HA → MQTT → SLS → HTTP → ESPHome → MQTT → HA</li>
    </ul>
  </div>
</section>

<section id="critical">
  <h2>⚠️ КРИТИЧЕСКИ ВАЖНО: Регистр букв</h2>
  
  <div class="critical">
    <strong>🚨 Это главная проблема, на которой ломается 90% интеграций!</strong>
    <br><br>
    MQTT чувствителен к регистру. <code>ZigbeeGate</code> и <code>ZIgbeeGate</code> — это <strong>два разных топика</code>. То же самое с именами объектов: <code>detskaya_WLED</code> и <code>detskaya_wled</code> — это разные объекты в SLS.
  </div>

  <h3>Правила, которые нужно соблюдать:</h3>
  
  <ol>
    <li><strong>Все имена объектов в SLS — строчными буквами</strong> (например, <code>detskaya_wled.brightness</code>, а не <code>detskaya_WLED.brightness</code>). Это потому что HA при MQTT Discovery генерирует топики в нижнем регистре.</li>
    
    <li><strong>Системный префикс SLS — точно как в настройках</strong>. Посмотри в SLS → Settings → MQTT → "Системный префикс топика". У тебя это <code>[имя твоего SLS шлюза/или корневого топика без скобок]</code> (с маленькой <code>i</code> во втором символе!). Используй его везде.</li>
    
    <li><strong>Имена компонентов ESPHome — только латиница в нижнем регистре</strong>. Например, <code>strip</code>, <code>effect_speed</code>, а не <code>Strip</code> или <code>EffectSpeed</code>.</li>
    
    <li><strong>Топики MQTT должны совпадать везде</strong>: в init.lua, в скриптах SLS, в конфиге HA, в MQTT Explorer. Любое расхождение в регистре = неработающая интеграция.</li>
  </ol>

  <h3>Пример правильной иерархии имён:</h3>
  <pre><code>SLS объект:      detskaya_wled.brightness       (строчные)
MQTT топик:      [имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.brightness
MQTT /set:       [имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.brightness/set
HA command:      [имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.brightness/set
HA state:        [имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.brightness
ESPHome entity:  strip                          (латиница, строчные)</code></pre>

  <div class="warning">
    <strong>⚠️ Если у тебя уже есть объекты с разным регистром</strong> — удали дубликаты в SLS и приведи всё к единому регистру. Иначе будут "фантомные" объекты, которые появляются в MQTT, но не имеют скриптов.
  </div>
</section>

<section id="features">
  <h2>Критические особенности ESPHome REST API v3</h2>
  
  <div class="warning">
    <strong>⚠️ Внимание!</strong> Эти особенности не очевидны из документации и являются результатом реального тестирования. Их игнорирование приводит к ошибкам 404, 411 и молчаливому игнорированию команд.
  </div>

  <h3>1. Content-Length обязателен для POST</h3>
  <p>Любой POST-запрос должен содержать заголовок <code>Content-Length</code>, иначе ESPHome вернёт ошибку <code>411 Length Required</code>. Это требование ESP-IDF HTTP-сервера.</p>
  
  <pre><code><span class="comment">-- Правильно:</span>
<span class="keyword">local</span> headers = <span class="string">"Content-Length: 2\r\n"</span>
<span class="keyword">local</span> body = <span class="string">"{}"</span>

<span class="comment">-- Неправильно (вернёт 411):</span>
http.request(url, <span class="string">"POST"</span>)</code></pre>

  <h3>2. Параметры в URL, а не в теле запроса</h3>
  <p>ESPHome web_server <strong>игнорирует JSON-тело</strong> для команд управления. Все параметры передаются через query-строку URL:</p>
  
  <pre><code><span class="comment">-- Правильно (параметр brightness в URL):</span>
<span class="keyword">local</span> url = <span class="string">"http://&lt;IP&gt;/light/strip/turn_on?brightness=150"</span>

<span class="comment">-- Неправильно (параметр в JSON-теле — будет проигнорирован):</span>
<span class="keyword">local</span> body = <span class="string">'{"brightness":150}'</span></code></pre>

  <h3>3. Только латиница в именах компонентов</h3>
  <p>REST API не поддерживает кириллицу в URL. Все поля <code>name:</code> в YAML должны быть на латинице:</p>
  
  <pre><code><span class="comment"># Правильно:</span>
light:
  - platform: esp32_rmt_led_strip
    name: <span class="string">"strip"</span>

<span class="comment"># Неправильно (вернёт 404):</span>
light:
  - platform: esp32_rmt_led_strip
    name: <span class="string">"Лента"</span></code></pre>

  <h3>4. Пустое тело запроса</h3>
  <p>Для POST-запросов используется пустой JSON <code>{}</code> с <code>Content-Length: 2</code>. Это обходной путь для бага парсера ESPHome web_server v3.</p>

  <h3>5. Обязательный параметр color_mode для RGB</h3>
  <p>При установке цвета необходимо указывать <code>color_mode=rgb</code>, иначе ESPHome может игнорировать значения R, G, B (известный баг #5750).</p>
</section>

<section id="components">
  <h2>Управление компонентами ESPHome</h2>

  <h3>Light (Свет)</h3>
  <h4>YAML конфиг:</h4>
  <pre><code>light:
  - platform: esp32_rmt_led_strip
    name: <span class="string">"strip"</span>
    id: strip
    pin: GPIO4
    num_leds: <span class="number">287</span>
    chipset: WS2812
    channel_colors: RGB</code></pre>

  <h4>Включение с яркостью:</h4>
  <pre><code><span class="keyword">local</span> url = <span class="string">"http://&lt;IP&gt;/light/strip/turn_on?brightness=150"</span>
http.request(url, <span class="string">"POST"</span>, <span class="string">"Content-Length: 2\r\n"</span>, <span class="string">"{}"</span>)</code></pre>

  <h4>Выключение:</h4>
  <pre><code><span class="keyword">local</span> url = <span class="string">"http://&lt;IP&gt;/light/strip/turn_off"</span>
http.request(url, <span class="string">"POST"</span>, <span class="string">"Content-Length: 2\r\n"</span>, <span class="string">"{}"</span>)</code></pre>

  <h4>Установка эффекта:</h4>
  <pre><code><span class="keyword">local</span> effect = <span class="string">"Matrix%20Rain"</span>  <span class="comment">-- Пробелы заменяем на %20</span>
<span class="keyword">local</span> url = <span class="string">"http://&lt;IP&gt;/light/strip/turn_on?effect="</span> .. effect
http.request(url, <span class="string">"POST"</span>, <span class="string">"Content-Length: 2\r\n"</span>, <span class="string">"{}"</span>)</code></pre>

  <h4>Установка цвета RGB:</h4>
  <pre><code><span class="comment">-- Обязательно color_mode=rgb!</span>
<span class="keyword">local</span> url = <span class="string">"http://&lt;IP&gt;/light/strip/turn_on?color_mode=rgb&amp;r=255&amp;g=0&amp;b=0"</span>
http.request(url, <span class="string">"POST"</span>, <span class="string">"Content-Length: 2\r\n"</span>, <span class="string">"{}"</span>)</code></pre>

  <h3>Number (Числовое значение)</h3>
  <h4>YAML конфиг:</h4>
  <pre><code>number:
  - platform: template
    name: <span class="string">"effect_speed"</span>
    id: effect_speed
    min_value: <span class="number">1</span>
    max_value: <span class="number">100</span>
    step: <span class="number">1</span>
    initial_value: <span class="number">20</span>
    mode: slider</code></pre>

  <h4>Установка значения:</h4>
  <pre><code><span class="keyword">local</span> speed = <span class="number">50</span>
<span class="keyword">local</span> url = <span class="string">"http://&lt;IP&gt;/number/effect_speed/set?value="</span> .. speed
http.request(url, <span class="string">"POST"</span>, <span class="string">"Content-Length: 2\r\n"</span>, <span class="string">"{}"</span>)</code></pre>

  <h3>Switch (Переключатель)</h3>
  <h4>YAML конфиг:</h4>
  <pre><code>switch:
  - platform: gpio
    name: <span class="string">"relay"</span>
    pin: GPIO5</code></pre>

  <h4>Включение/выключение:</h4>
  <pre><code><span class="comment">-- Включение:</span>
<span class="keyword">local</span> url = <span class="string">"http://&lt;IP&gt;/switch/relay/turn_on"</span>
http.request(url, <span class="string">"POST"</span>, <span class="string">"Content-Length: 2\r\n"</span>, <span class="string">"{}"</span>)

<span class="comment">-- Выключение:</span>
<span class="keyword">local</span> url = <span class="string">"http://&lt;IP&gt;/switch/relay/turn_off"</span>
http.request(url, <span class="string">"POST"</span>, <span class="string">"Content-Length: 2\r\n"</span>, <span class="string">"{}"</span>)</code></pre>

  <h3>Другие компоненты (кратко)</h3>
  <table>
    <tr><th>Компонент</th><th>Действие</th><th>URL</th></tr>
    <tr><td>Fan</td><td>Включить</td><td><code>/fan/ceiling_fan/turn_on?speed_level=2</code></td></tr>
    <tr><td>Cover</td><td>Открыть</td><td><code>/cover/blinds/open</code></td></tr>
    <tr><td>Cover</td><td>Позиция</td><td><code>/cover/blinds/set?position=0.5</code></td></tr>
    <tr><td>Select</td><td>Опция</td><td><code>/select/mode/set?option=auto</code></td></tr>
    <tr><td>Button</td><td>Нажать</td><td><code>/button/reboot/press</code></td></tr>
    <tr><td>Sensor</td><td>Читать (GET)</td><td><code>/sensor/temperature</code></td></tr>
  </table>
</section>

<section id="sls">
  <h2>Интеграция с SLS шлюзом</h2>

  <h3>API SLS для работы с объектами</h3>
  
  <div class="info">
    <strong>Важно:</strong> В SLS объекты хранятся <strong>только в памяти</strong> и не сохраняются во flash. Поэтому инициализация делается в <code>init.lua</code>.
  </div>

  <h4>Основные функции:</h4>
  <pre><code><span class="comment">-- Создание объекта с типом (если не существует — создаст)</span>
obj.setType(<span class="string">"detskaya_wled.brightness"</span>, <span class="string">"INT"</span>)

<span class="comment">-- Установка значения</span>
obj.set(<span class="string">"detskaya_wled.brightness"</span>, <span class="number">150</span>)

<span class="comment">-- Получение значения (МОЖЕТ ВЕРНУТЬ NIL!)</span>
<span class="keyword">local</span> val = obj.get(<span class="string">"detskaya_wled.brightness"</span>)

<span class="comment">-- Привязка скрипта с флагом W (Write)</span>
<span class="comment">-- true = запуск скрипта при каждой записи</span>
obj.setScript(<span class="string">"detskaya_wled.brightness"</span>, <span class="string">"detskaya_wled.lua"</span>, <span class="keyword">true</span>)

<span class="comment">-- Включение флага M (MQTT Notify)</span>
<span class="comment">-- true = публикация в MQTT, false = без MQTT</span>
obj.setShare(<span class="string">"detskaya_wled.brightness"</span>, <span class="keyword">true</span>, <span class="keyword">false</span>)

<span class="comment">-- Удаление объекта</span>
obj.remove(<span class="string">"detskaya_wled.brightness"</span>)</code></pre>

  <h4>Флаги объектов:</h4>
  <table>
    <tr><th>Флаг</th><th>Название</th><th>Описание</th></tr>
    <tr><td><strong>M</strong></td><td>MQTT Notify</td><td>При изменении объекта значение публикуется в MQTT. SLS <strong>автоматически подписывается</strong> на топик <code>.../obj/имя/set</code> и обновляет объект при получении команды.</td></tr>
    <tr><td><strong>W</strong></td><td>Write</td><td>Скрипт вызывается при каждой записи, даже если значение не изменилось</td></tr>
    <tr><td><strong>A</strong></td><td>ACK</td><td>Флаг обратной связи (для предотвращения циклов)</td></tr>
    <tr><td><strong>N</strong></td><td>Network</td><td>Синхронизация между шлюзами SLS в локальной сети</td></tr>
  </table>

  <div class="critical">
    <strong>🚨 Критически важно!</strong> В SLS значение объекта передаётся через <code>obj.get("имя")</code>, а НЕ через глобальную переменную <code>value</code>. Последний вернёт <code>nil</code>!
  </div>

  <pre><code><span class="comment">-- Правильно:</span>
<span class="keyword">local</span> brightness = obj.get(<span class="string">"detskaya_wled.brightness"</span>)
<span class="keyword">local</span> num = tonumber(brightness) <span class="keyword">or</span> <span class="number">0</span>

<span class="comment">-- Неправильно (вернёт nil):</span>
<span class="keyword">local</span> brightness = value</code></pre>
</section>

<section id="init">
  <h2>init.lua — Обязательная инициализация</h2>

  <div class="critical">
    <strong>🚨 Главное правило:</strong> Все имена объектов — <strong>строчными буквами</strong>!
    HA публикует в MQTT топики в нижнем регистре. Если в SLS объект называется с большой буквы — 
    SLS не свяжет его с MQTT-топиком и создаст дубликат.
  </div>

  <div class="info">
    <strong>Как работает флаг M (MQTT Notify):</strong>
    <ul>
      <li><code>obj.setShare(name, true, false)</code> устанавливает флаг M</li>
      <li>SLS <strong>автоматически публикует</strong> изменения в топик <code>.../obj/имя</code></li>
      <li>SLS <strong>автоматически подписывается</strong> на топик <code>.../obj/имя/set</code></li>
      <li>При получении сообщения в <code>/set</code> — SLS сам обновляет объект</li>
      <li>Никакого явного <code>mqtt.sub()</code> не нужно!</li>
    </ul>
  </div>

  <h3>init.lua (финальная версия)</h3>
  <pre><code><span class="comment">-- init.lua — Инициализация SLS шлюза</span>
<span class="comment">-- ВАЖНО: все имена объектов — строчными буквами!</span>

<span class="comment">-- 1. Zigbee скрипты</span>
obj.setType(<span class="string">"isDay"</span>, <span class="string">"BOOL"</span>)
scripts.run(<span class="string">"day_night.lua"</span>)
scripts.setTimer(<span class="string">"day_night.lua"</span>, <span class="number">60</span>)

<span class="comment">-- 2. Конфигурация ESPHome объектов (ВСЕ МАЛЕНЬКИМИ БУКВАМИ!)</span>
<span class="keyword">local</span> devices = {
    {name = <span class="string">"detskaya_wled.brightness"</span>,    type = <span class="string">"INT"</span>,  default = <span class="number">255</span>,  script = <span class="string">"detskaya_wled.lua"</span>},
    {name = <span class="string">"detskaya_wled.effect"</span>,         type = <span class="string">"STR"</span>,  default = <span class="string">"None"</span>, script = <span class="string">"detskaya_wled_effect.lua"</span>},
    {name = <span class="string">"detskaya_wled.effect_speed"</span>,   type = <span class="string">"INT"</span>,  default = <span class="number">20</span>,   script = <span class="string">"detskaya_wled_speed.lua"</span>},
    {name = <span class="string">"detskaya_wled.color"</span>,          type = <span class="string">"STR"</span>,  default = <span class="string">"FFFFFF"</span>, script = <span class="string">"detskaya_wled_color.lua"</span>}
}

<span class="comment">-- 3. Инициализация и настройка</span>
<span class="keyword">for</span> _, dev <span class="keyword">in</span> ipairs(devices) <span class="keyword">do</span>
    <span class="keyword">local</span> current = obj.get(dev.name)
    
    <span class="comment">-- Создаём объект ТОЛЬКО если его нет</span>
    <span class="keyword">if</span> current == <span class="keyword">nil</span> <span class="keyword">then</span>
        obj.setType(dev.name, dev.type)
        obj.set(dev.name, dev.default)
    <span class="keyword">end</span>
    
    <span class="comment">-- ПРИНУДИТЕЛЬНО устанавливаем скрипт и флаги при каждой загрузке</span>
    obj.setScript(dev.name, dev.script, <span class="keyword">true</span>)   <span class="comment">-- флаг W (Write)</span>
    obj.setShare(dev.name, <span class="keyword">true</span>, <span class="keyword">false</span>)          <span class="comment">-- флаг M (MQTT Notify)</span>
<span class="keyword">end</span>

<span class="comment">-- 4. Опционально: таймер для восстановления из MQTT при старте</span>
<span class="comment">-- scripts.setTimer("mqtt_restore.lua", 5)</span></code></pre>

  <div class="success">
    <strong>✅ Что делает этот скрипт:</strong>
    <ul>
      <li>Создаёт объекты с правильными типами и дефолтами</li>
      <li>Привязывает скрипты (флаг W) — при каждой загрузке</li>
      <li>Включает MQTT (флаг M) — SLS сам подписывается на <code>/set</code> топики</li>
      <li>HA → MQTT → SLS → ESPHome работает без всяких мостов</li>
    </ul>
  </div>

  <div class="warning">
    <strong>⚠️ Если у тебя уже есть дубликаты с разным регистром</strong> — удали в SLS объекты с большой буквой 
    (<code>detskaya_WLED.*</code>), оставь только строчные (<code>detskaya_wled.*</code>). 
    Иначе HA будет обновлять "фантомные" объекты без скриптов.
  </div>
</section>

<section id="ha">
  <h2>Интеграция с Home Assistant через MQTT</h2>

  <div class="info">
    <strong>Предварительные требования:</strong>
    <ul>
      <li>MQTT брокер (Mosquitto) установлен и работает (например, на <code>192.168.1.x - IP твоего MQTT брокера</code>)</li>
      <li>SLS подключён к MQTT брокеру (Settings → MQTT → Enable)</li>
      <li>HA подключён к тому же MQTT брокеру (Settings → Devices & Services → MQTT)</li>
      <li>В SLS у объектов установлен флаг M (MQTT Notify)</li>
    </ul>
  </div>

  <h3>Способ 1: Через YAML в configuration.yaml (рекомендуется)</h3>
  
  <p>Добавь в <code>configuration.yaml</code>:</p>
  
  <pre><code>mqtt:
  light:
    - name: <span class="string">"Подсветка детская"</span>
      unique_id: <span class="string">"wled_podsvetka_detskaya"</span>
      
      <span class="comment"># Основные топики</span>
      command_topic: <span class="string">"[имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.brightness/set"</span>
      state_topic: <span class="string">"[имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.brightness"</span>
      
      <span class="comment"># Яркость</span>
      brightness_command_topic: <span class="string">"[имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.brightness/set"</span>
      brightness_state_topic: <span class="string">"[имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.brightness"</span>
      brightness_scale: <span class="number">255</span>
      
      <span class="comment"># Цвет RGB</span>
      rgb_command_topic: <span class="string">"[имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.color/set"</span>
      rgb_state_topic: <span class="string">"[имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.color"</span>
      rgb_command_template: <span class="string">"{{ '%02X%02X%02X' | format(red, green, blue) }}"</span>
      rgb_value_template: <span class="string">"{{ value[0:2]|int(base=16) }},{{ value[2:4]|int(base=16) }},{{ value[4:6]|int(base=16) }}"</span>
      
      <span class="comment"># Эффекты</span>
      effect_command_topic: <span class="string">"[имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.effect/set"</span>
      effect_state_topic: <span class="string">"[имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.effect"</span>
      effect_list:
        - <span class="string">"None"</span>
        - <span class="string">"Random"</span>
        - <span class="string">"Strobe"</span>
        - <span class="string">"Flicker"</span>
        - <span class="string">"Pulse"</span>
        - <span class="string">"Rainbow"</span>
        - <span class="string">"Matrix Rain"</span>
        - <span class="string">"Fire"</span>
      
      <span class="comment"># Состояние ON/OFF</span>
      state_value_template: <span class="string">"{% if value | int > 0 %}ON{% else %}OFF{% endif %}"</span>
      payload_on: <span class="string">"255"</span>
      payload_off: <span class="string">"0"</span>
      on_command_type: <span class="string">"brightness"</span>  <span class="comment"># ВАЖНО: отправлять реальную яркость, а не ON/OFF</span>
      
      <span class="comment"># Настройки</span>
      optimistic: <span class="keyword">true</span>
      qos: <span class="number">0</span>
      schema: <span class="string">"basic"</span></code></pre>

  <div class="critical">
    <strong>🚨 Ключевые параметры, которые часто забывают:</strong>
    <ul>
      <li><code>on_command_type: brightness</code> — без этого HA будет отправлять только 0/255 вместо промежуточных значений яркости</li>
      <li><code>optimistic: true</code> — HA не ждёт подтверждения и сразу обновляет UI</li>
      <li><code>payload_on: "255"</code> и <code>payload_off: "0"</code> — должны быть числами, а не ON/OFF</li>
      <li><strong>Регистр во всех топиках</strong> должен точно совпадать с тем, что в SLS!</li>
    </ul>
  </div>

  <h3>Способ 2: Через веб-интерфейс HA</h3>
  
  <ol>
    <li>Settings → Devices & Services → MQTT → Configure → Add Entity → Light</li>
    <li><strong>Name:</strong> <code>Подсветка детская</code></li>
    <li><strong>Unique ID:</strong> <code>wled_podsvetka_detskaya</code></li>
    <li><strong>Schema:</strong> <code>Default schema</code></li>
    <li><strong>Command topic:</strong> <code>[имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.brightness/set</code></li>
    <li><strong>State topic:</strong> <code>[имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.brightness</code></li>
    <li><strong>Тип команды ВКЛ:</strong> <code>Яркость</code> (Brightness) — <strong>ОБЯЗАТЕЛЬНО!</strong></li>
  </ol>

  <h4>Настройки яркости (Brightness settings):</h4>
  <ul>
    <li><strong>Brightness command topic:</strong> <code>[имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.brightness/set</code></li>
    <li><strong>Brightness state topic:</strong> <code>[имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.brightness</code></li>
    <li><strong>Brightness scale:</strong> <code>255</code></li>
  </ul>

  <h4>Настройки RGB (RGB color settings):</h4>
  <ul>
    <li><strong>RGB command topic:</strong> <code>[имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.color/set</code></li>
    <li><strong>RGB state topic:</strong> <code>[имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.color</code></li>
    <li><strong>RGB command template:</strong> <code>{{ '%02X%02X%02X' | format(red, green, blue) }}</code></li>
    <li><strong>RGB value template:</strong> <code>{{ value[0:2]|int(base=16) }},{{ value[2:4]|int(base=16) }},{{ value[4:6]|int(base=16) }}</code></li>
  </ul>

  <h4>Настройки эффекта (Effect settings):</h4>
  <ul>
    <li><strong>Effect command topic:</strong> <code>[имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.effect/set</code></li>
    <li><strong>Effect state topic:</strong> <code>[имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.effect</code></li>
    <li><strong>Список эффектов:</strong> добавь через "+ Добавить": None, Random, Strobe, Flicker, Pulse, Rainbow, Matrix Rain, Fire</li>
  </ul>

  <h4>Other settings:</h4>
  <ul>
    <li><strong>State value template:</strong> <code>{% if value | int > 0 %}ON{% else %}OFF{% endif %}</code></li>
    <li><strong>Payload on:</strong> <code>255</code></li>
    <li><strong>Payload off:</strong> <code>0</code></li>
    <li><strong>Optimistic:</strong> ✅ включи</li>
    <li><strong>Retain:</strong> ❌ выключи</li>
  </ul>

  <h3>Проверка работы</h3>
  <ol>
    <li>Перезагрузи HA после изменения конфигурации</li>
    <li>В HA должно появиться устройство "Подсветка детская"</li>
    <li>Подвигай ползунок яркости — в SLS → Objects должно измениться значение</li>
    <li>Лента должна отреагировать</li>
    <li>Выбери цвет в color picker — лента должна изменить цвет</li>
    <li>Выбери эффект из списка — эффект должен запуститься</li>
  </ol>

  <div class="warning">
    <strong>⚠️ Если команды не доходят до SLS:</strong>
    <ol>
      <li>Проверь в MQTT Explorer, публикует ли HA в правильный топик</li>
      <li>Проверь регистр — <code>ZigbeeGate</code> ≠ <code>ZIgbeeGate</code></li>
      <li>Проверь, что у объектов в SLS стоит флаг M</li>
      <li>Проверь, что имена объектов в SLS строчными буквами</li>
    </ol>
  </div>
</section>

<section id="example">
  <h2>Полный пример: управление LED-лентой</h2>

  <h3>YAML конфиг ESPHome</h3>
  <pre><code>esphome:
  name: wled-c3

esp32:
  variant: esp32c3
  framework:
    type: esp-idf

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

web_server:
  port: <span class="number">80</span>
  version: <span class="number">3</span>

light:
  - platform: esp32_rmt_led_strip
    name: <span class="string">"strip"</span>
    id: strip
    pin: GPIO4
    num_leds: <span class="number">287</span>
    chipset: WS2812
    channel_colors: RGB
    effects:
      - addressable_lambda:
          name: <span class="string">"Matrix Rain"</span>
          update_interval: 50ms
          lambda: |-
            <span class="comment">// Код эффекта...</span>
      - addressable_lambda:
          name: <span class="string">"Fire"</span>
          update_interval: 40ms
          lambda: |-
            <span class="comment">// Код эффекта...</span>

number:
  - platform: template
    name: <span class="string">"effect_speed"</span>
    id: effect_speed
    min_value: <span class="number">1</span>
    max_value: <span class="number">100</span>
    step: <span class="number">1</span>
    initial_value: <span class="number">20</span>
    mode: slider</code></pre>

  <h3>Скрипты SLS</h3>
  
  <h4>Файл: <code>detskaya_wled.lua</code> (управление яркостью)</h4>
  <pre><code><span class="keyword">local</span> ip_esp = <span class="string">"[IP адрес твоего ESPHome устройства без скобок]"</span>
<span class="keyword">local</span> entity = <span class="string">"strip"</span>

<span class="keyword">local</span> raw_val = obj.get(<span class="string">"detskaya_wled.brightness"</span>)
<span class="keyword">local</span> brightness = tonumber(raw_val) <span class="keyword">or</span> <span class="number">0</span>

<span class="keyword">if</span> brightness &lt; <span class="number">0</span> <span class="keyword">then</span> brightness = <span class="number">0</span> <span class="keyword">end</span>

<span class="keyword">local</span> headers = <span class="string">"Content-Length: 2\r\n"</span>
<span class="keyword">local</span> body = <span class="string">"{}"</span>

<span class="keyword">if</span> brightness &gt; <span class="number">0</span> <span class="keyword">then</span>
    <span class="keyword">local</span> url = <span class="string">"http://"</span> .. ip_esp .. <span class="string">"/light/"</span> .. entity .. <span class="string">"/turn_on?brightness="</span> .. brightness
    http.request(url, <span class="string">"POST"</span>, headers, body)
<span class="keyword">else</span>
    <span class="keyword">local</span> url = <span class="string">"http://"</span> .. ip_esp .. <span class="string">"/light/"</span> .. entity .. <span class="string">"/turn_off"</span>
    http.request(url, <span class="string">"POST"</span>, headers, body)
<span class="keyword">end</span></code></pre>

  <h4>Файл: <code>detskaya_wled_effect.lua</code> (выбор эффекта)</h4>
  <pre><code><span class="keyword">local</span> ip_esp = <span class="string">"[IP адрес твоего ESPHome устройства без скобок]"</span>
<span class="keyword">local</span> entity = <span class="string">"strip"</span>

<span class="keyword">local</span> raw_effect = obj.get(<span class="string">"detskaya_wled.effect"</span>)
<span class="keyword">if</span> raw_effect == <span class="keyword">nil</span> <span class="keyword">or</span> raw_effect == <span class="string">""</span> <span class="keyword">then</span> <span class="keyword">return</span> <span class="keyword">end</span>

<span class="keyword">local</span> effect_name = string.gsub(raw_effect, <span class="string">" "</span>, <span class="string">"%%20"</span>)
<span class="keyword">local</span> headers = <span class="string">"Content-Length: 2\r\n"</span>
<span class="keyword">local</span> body = <span class="string">"{}"</span>
<span class="keyword">local</span> url = <span class="string">"http://"</span> .. ip_esp .. <span class="string">"/light/"</span> .. entity .. <span class="string">"/turn_on?effect="</span> .. effect_name
http.request(url, <span class="string">"POST"</span>, headers, body)</code></pre>

  <h4>Файл: <code>detskaya_wled_speed.lua</code> (скорость эффекта)</h4>
  <pre><code><span class="keyword">local</span> ip_esp = <span class="string">"[IP адрес твоего ESPHome устройства без скобок]"</span>
<span class="keyword">local</span> raw_speed = obj.get(<span class="string">"detskaya_wled.effect_speed"</span>)
<span class="keyword">local</span> speed = tonumber(raw_speed) <span class="keyword">or</span> <span class="number">20</span>
<span class="keyword">if</span> speed &lt; <span class="number">1</span> <span class="keyword">then</span> speed = <span class="number">1</span> <span class="keyword">end</span>
<span class="keyword">if</span> speed &gt; <span class="number">100</span> <span class="keyword">then</span> speed = <span class="number">100</span> <span class="keyword">end</span>

<span class="keyword">local</span> headers = <span class="string">"Content-Length: 2\r\n"</span>
<span class="keyword">local</span> body = <span class="string">"{}"</span>
<span class="keyword">local</span> url = <span class="string">"http://"</span> .. ip_esp .. <span class="string">"/number/effect_speed/set?value="</span> .. speed
http.request(url, <span class="string">"POST"</span>, headers, body)</code></pre>

  <h4>Файл: <code>detskaya_wled_color.lua</code> (управление цветом)</h4>
  <pre><code><span class="keyword">local</span> ip_esp = <span class="string">"[IP адрес твоего ESPHome устройства без скобок]"</span>
<span class="keyword">local</span> entity = <span class="string">"strip"</span>

<span class="keyword">local</span> hex = obj.get(<span class="string">"detskaya_wled.color"</span>)
<span class="keyword">if</span> hex == <span class="keyword">nil</span> <span class="keyword">or</span> hex == <span class="string">""</span> <span class="keyword">then</span> <span class="keyword">return</span> <span class="keyword">end</span>

hex = string.upper(hex)
hex = string.gsub(hex, <span class="string">"#"</span>, <span class="string">""</span>)
<span class="keyword">if</span> string.len(hex) ~= <span class="number">6</span> <span class="keyword">then</span> <span class="keyword">return</span> <span class="keyword">end</span>

<span class="keyword">local</span> r = tonumber(string.sub(hex, <span class="number">1</span>, <span class="number">2</span>), <span class="number">16</span>) <span class="keyword">or</span> <span class="number">0</span>
<span class="keyword">local</span> g = tonumber(string.sub(hex, <span class="number">3</span>, <span class="number">4</span>), <span class="number">16</span>) <span class="keyword">or</span> <span class="number">0</span>
<span class="keyword">local</span> b = tonumber(string.sub(hex, <span class="number">5</span>, <span class="number">6</span>), <span class="number">16</span>) <span class="keyword">or</span> <span class="number">0</span>

<span class="keyword">local</span> headers = <span class="string">"Content-Length: 2\r\n"</span>
<span class="keyword">local</span> body = <span class="string">"{}"</span>
<span class="keyword">local</span> url = <span class="string">"http://"</span> .. ip_esp .. <span class="string">"/light/"</span> .. entity .. <span class="string">"/turn_on?color_mode=rgb&amp;r="</span> .. r .. <span class="string">"&amp;g="</span> .. g .. <span class="string">"&amp;b="</span> .. b
http.request(url, <span class="string">"POST"</span>, headers, body)</code></pre>

  <h3>Итоговая таблица объектов SLS</h3>
  <table>
    <tr><th>Имя объекта</th><th>Тип</th><th>Default</th><th>Скрипт</th><th>Флаги</th></tr>
    <tr><td><code>detskaya_wled.brightness</code></td><td>INT</td><td>255</td><td>detskaya_wled.lua</td><td>MW</td></tr>
    <tr><td><code>detskaya_wled.effect</code></td><td>STR</td><td>None</td><td>detskaya_wled_effect.lua</td><td>MW</td></tr>
    <tr><td><code>detskaya_wled.effect_speed</code></td><td>INT</td><td>20</td><td>detskaya_wled_speed.lua</td><td>MW</td></tr>
    <tr><td><code>detskaya_wled.color</code></td><td>STR</td><td>FFFFFF</td><td>detskaya_wled_color.lua</td><td>MW</td></tr>
  </table>
</section>

<section id="colors">
  <h2>Управление цветом RGB через HEX-код</h2>
  
  <p>Вместо трёх отдельных объектов для R, G, B используется один универсальный объект с HEX-кодом цвета.</p>

  <h3>Шпаргалка популярных цветов</h3>
  <table>
    <tr><th>Цвет</th><th>HEX-код</th></tr>
    <tr><td>⚪ Белый</td><td><code>FFFFFF</code></td></tr>
    <tr><td>⚫ Чёрный</td><td><code>000000</code></td></tr>
    <tr><td>🔴 Красный</td><td><code>FF0000</code></td></tr>
    <tr><td>🟢 Зелёный</td><td><code>00FF00</code></td></tr>
    <tr><td>🔵 Синий</td><td><code>0000FF</code></td></tr>
    <tr><td>🟡 Жёлтый</td><td><code>FFFF00</code></td></tr>
    <tr><td>🩵 Голубой (Cyan)</td><td><code>00FFFF</code></td></tr>
    <tr><td>🟣 Пурпурный (Magenta)</td><td><code>FF00FF</code></td></tr>
    <tr><td>🟠 Оранжевый</td><td><code>FFA500</code></td></tr>
    <tr><td>🌸 Розовый</td><td><code>FFC0CB</code></td></tr>
    <tr><td>🟪 Фиолетовый</td><td><code>800080</code></td></tr>
    <tr><td>🕯️ Тёплый белый</td><td><code>FFE4B5</code></td></tr>
    <tr><td>❄️ Холодный белый</td><td><code>F0F8FF</code></td></tr>
  </table>

  <div class="info">
    <strong>💡 Совет:</strong> Можно вводить с <code>#</code> или без — скрипт поймёт оба варианта: <code>#FF0000</code> и <code>FF0000</code> работают одинаково.
  </div>
</section>

<section id="debug">
  <h2>Отладка</h2>

  <h3>Проверка доступности ESPHome</h3>
  <p>Откройте в браузере: <code>http://&lt;IP_ESP&gt;/light/strip</code></p>
  <p>Должен вернуться JSON:</p>
  <pre><code>{
  <span class="string">"id"</span>: <span class="string">"light/strip"</span>,
  <span class="string">"state"</span>: <span class="string">"OFF"</span>,
  <span class="string">"brightness"</span>: <span class="number">255</span>,
  <span class="string">"color"</span>: {<span class="string">"r"</span>: <span class="number">255</span>, <span class="string">"g"</span>: <span class="number">255</span>, <span class="string">"b"</span>: <span class="number">255</span>}
}</code></pre>

  <h3>Проверка запроса через curl</h3>
  <pre><code><span class="comment"># Включение с яркостью</span>
curl -X POST -H <span class="string">"Content-Length: 2"</span> -d <span class="string">"{}"</span> <span class="string">"http://&lt;IP&gt;/light/strip/turn_on?brightness=150"</span>

<span class="comment"># Выключение</span>
curl -X POST -H <span class="string">"Content-Length: 2"</span> -d <span class="string">"{}"</span> <span class="string">"http://&lt;IP&gt;/light/strip/turn_off"</span>

<span class="comment"># Установка цвета</span>
curl -X POST -H <span class="string">"Content-Length: 2"</span> -d <span class="string">"{}"</span> <span class="string">"http://&lt;IP&gt;/light/strip/turn_on?color_mode=rgb&amp;r=255&amp;g=0&amp;b=0"</span></code></pre>

  <h3>Проверка MQTT потока</h3>
  <pre><code><span class="comment"># Подписка на все топики SLS</span>
mosquitto_sub -h [IP твоего MQTT брокера без скобок] -t "[имя твоего SLS шлюза/или корневого топика без скобок]/obj/#" -v

<span class="comment"># Публикация команды</span>
mosquitto_pub -h [IP твоего MQTT брокера без скобок] -t "[имя твоего SLS шлюза/или корневого топика без скобок]/obj/detskaya_wled.brightness/set" -m "150"</code></pre>

  <h3>Таблица ошибок</h3>
  <table>
    <tr><th>Ошибка</th><th>Причина</th><th>Решение</th></tr>
    <tr><td><code>404 Not Found</code></td><td>Неправильное имя компонента или кириллица</td><td>Проверьте имя в YAML (только латиница)</td></tr>
    <tr><td><code>411 Length Required</code></td><td>Отсутствует Content-Length</td><td>Добавьте <code>Content-Length: 2</code></td></tr>
    <tr><td><code>Error -1</code> в SLS</td><td>HTTP не дошёл до ESPHome</td><td>Проверьте IP и доступность устройства</td></tr>
    <tr><td>HA не управляет</td><td>Неправильный регистр в топиках</td><td>Приведи все имена к строчным буквам</td></tr>
    <tr><td>HA шлёт только 0/255</td><td>on_command_type не brightness</td><td>Установи <code>on_command_type: brightness</code></td></tr>
    <tr><td>Дубликаты объектов</td><td>Разный регистр имён</td><td>Удали дубликаты, используй строчные буквы</td></tr>
    <tr><td>TemplateSyntaxError</td><td>Многострочный шаблон в UI</td><td>Используй YAML или однострочный шаблон</td></tr>
    <tr><td>ValueError: invalid literal for int()</td><td>rgb_value_template возвращает список</td><td>Убери квадратные скобки в шаблоне</td></tr>
  </table>
</section>

<section id="faq">
  <h2>Часто задаваемые вопросы</h2>

  <h4>Q: Почему нельзя использовать кириллицу в именах компонентов ESPHome?</h4>
  <p>A: ESPHome REST API не поддерживает URL-encoding кириллицы. Все имена должны быть на латинице.</p>

  <h4>Q: Почему POST без Content-Length возвращает 411?</h4>
  <p>A: Это требование ESP-IDF HTTP-сервера. Все POST-запросы должны содержать заголовок Content-Length.</p>

  <h4>Q: Почему JSON-тело игнорируется?</h4>
  <p>A: ESPHome web_server v3 читает параметры только из URL query-строки, не из тела запроса.</p>

  <h4>Q: Почему value в Lua-скрипте всегда nil?</h4>
  <p>A: В SLS значение объекта передаётся через <code>obj.get("имя_объекта")</code>, а не через глобальную переменную <code>value</code>.</p>

  <h4>Q: Почему объекты сбрасываются после перезагрузки SLS?</h4>
  <p>A: Объекты хранятся только в RAM. Используй init.lua с принудительной установкой скриптов и флагов при каждой загрузке.</p>

  <h4>Q: Почему HA отправляет только 0/255 вместо промежуточных значений?</h4>
  <p>A: Установи <code>on_command_type: brightness</code> в конфиге HA. Иначе HA отправляет payload_on/payload_off вместо реальной яркости.</p>

  <h4>Q: Почему создаются дубликаты объектов в SLS?</h4>
  <p>A: Из-за разного регистра. HA генерирует топики в нижнем регистре, а если в SLS объекты с большой буквы — создаются дубликаты. Решение: использовать строчные буквы везде.</p>

  <h4>Q: Почему команды из HA не доходят до SLS?</h4>
  <p>A: Проверь регистр букв. SLS автоматически подписывается на топики <code>/set</code> благодаря флагу M. Если регистр не совпадает — создаются дубликаты без скриптов.</p>

  <h4>Q: Как проверить, что MQTT работает?</h4>
  <p>A: Используй MQTT Explorer или <code>mosquitto_sub -t "[имя твоего SLS шлюза/или корневого топика без скобок]/obj/#" -v</code>. Меняй значение в SLS — должно появиться сообщение в MQTT.</p>

  <h4>Q: Можно ли управлять несколькими ESPHome устройствами?</h4>
  <p>A: Да. Создай отдельные объекты с разными префиксами (например, <code>kitchen_wled.*</code>, <code>bedroom_wled.*</code>) и отдельные скрипты с разными IP.</p>

  <h4>Q: Нужен ли MQTT для базовой работы?</h4>
  <p>A: Нет. SLS может управлять ESPHome напрямую через HTTP. MQTT нужен только для интеграции с Home Assistant.</p>
</section>

</main>

<footer>
  <p>📝 Документация создана на основе реального тестирования ESPHome 2026.9.1 + SLS 2026.08.30d5 + Home Assistant 2026.x</p>
  <p>🔧 Обновлено: 12 ноября 2026</p>
  <p>👤 Автор: @diemon24</p>
  <p>💡 Ключевые инсайты: регистр букв, флаг M автоматически подписывается на /set, on_command_type: brightness</p>
</footer>

</body>
</html>
