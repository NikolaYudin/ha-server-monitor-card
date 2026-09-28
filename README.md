<p align="center">
  <img src="https://img.shields.io/github/v/release/YOUR_NICK/ha-server-monitor-card?style=flat-square" alt="Release">
  <img src="https://img.shields.io/github/license/YOUR_NICK/ha-server-monitor-card?style=flat-square" alt="License">
  <img src="https://img.shields.io/badge/Home%20Assistant-2024.8%2B-41BDF5?style=flat-square&logo=homeassistant" alt="HA">
</p>

<h1 align="center">🖥️ Server Monitor PRO</h1>

<p align="center">
  Панель мониторинга Proxmox-инфраструктуры в стиле Windows 11:<br>
  4 колонки серверов, кнопки управления в шапке, фирменные цвета брендов<br>
  <i>Один YAML-файл · четыре зависимости из HACS · адаптив к теме</i>
</p>

<p align="center">
  <img src="screenshots/desktop.png" width="100%">
</p>

## ✨ Что внутри

- 🖥️ **4 сервера** — PVE, Xpenology, Home Assistant OS, Q4OS
- 🎨 **Фирменные цвета** — каждая колонка в айдентике бренда (PVE 🟠, XPE 🔵, HA 🔷, Q4OS 🟣)
- 🎛️ **Кнопки в шапке** — управление VM (Start/Stop/Reboot/Shutdown) прямо в заголовке, подтверждение для опасных действий
- 📊 **Графики** — CPU, RAM, Network I/O, Storage с компактным представлением 24 часа
- 🪟 **Стиль Windows 11** — скругления 8px, Segoe UI Variable, мягкие тени, полупрозрачные кнопки как в таскбаре
- 🌓 **Адаптив к теме** — поверхности через `var(--ha-card-background)`, корректно отображается в светлой и тёмной теме
- 🔒 **Подтверждения** — все shutdown/reboot требуют подтверждения, защита от случайного нажатия
- 📱 **Адаптивность** — 4 колонки на десктопе, стек на мобильных

## 🧩 Требования

- Home Assistant **2024.8+**
- HACS (Frontend): **mushroom-cards**, **card-mod**, **mini-graph-card**, **button-card**
- Интеграция с Proxmox VE (через HACS `proxmoxve` или кастомные сенсоры)
- Entity-сенсоры для каждой ВМ: status, uptime, cpu, memory, network, storage

## 🚀 Установка

1. HACS → Frontend → установите:
   - **Mushroom** (piitaya/lovelace-mushroom)
   - **card-mod** (thomasloven/lovelace-card-mod)
   - **mini-graph-card** (kalkih/mini-graph-card)
   - **button-card** (custom-cards/button-card)

2. Скопируйте `lovelace/server_monitor_card.yaml` в редактор дашборда:
   - Откройте дашборд → ⋮ → Редактировать → + Добавить карточку → Вручную
   - Вставьте YAML-код

3. Замените entity_id на свои:

   | ВМ | Требуемые сенсоры |
   |---|---|
   | **PVE** | `sensor.pve_status`, `sensor.pve_uptime`, `sensor.pve_cpu_usage`, `sensor.pve_memory_usage_percentage`, `sensor.storage_local_*`, `button.pve_stop/reboot/shutdown` |
   | **XPE** | `sensor.xpe_status`, `sensor.xpe_uptime`, `sensor.xpe_cpu_usage`, `sensor.xpe_memory_usage_percentage`, `sensor.xpe_network_input/output`, `sensor.vor0byshka_volume_*`, `button.nas_reboot/shutdown` |
   | **HAOS** | `sensor.haos_*_status/uptime/cpu/memory/network_*`, `button.haos_*_start/stop/reboot/shutdown` |
   | **Q4OS** | `sensor.q4os_*`, `button.q4os_start/stop/reboot/shutdown` |

4. Сохраните и обновите страницу (Ctrl+Shift+R).

## 🎨 Фирменные цвета

Каждый сервер получил свою айдентику через `border-left` и акцентные цвета графиков/кнопок:

| Сервер | Основной | Тёмный | Иконка |
|---|---|---|---|
| **Proxmox VE** | `#E57000` оранжевый | `#404040` | 🟠 |
| **Xpenology** | `#0066B3` синий | `#1A3050` | 🔵 |
| **Home Assistant** | `#41BDF5` голубой | `#005A87` | 🔷 |
| **Q4OS** | `#4456A0` индиго | `#2B3870` | 🟣 |

## 🛠 Как получить сенсоры Proxmox

### Вариант 1: HACS-интеграция ProxmoxVE
Установите [`proxmoxve`](https://github.com/dougiteixeira/proxmoxve) через HACS → добавьте integration → сенсоры создадутся автоматически.

### Вариант 2: REST-команды к Proxmox API
```yaml
rest:
  - resource: http://pve.local:8006/api2/json/nodes/pve/status
    headers:
      Authorization: !secret pve_token
    sensor:
      - name: "PVE CPU"
        value_template: "{{ value_json.data.cpu | float * 100 | round(1) }}"
        unit_of_measurement: "%"
```

### Вариант 3: Glances / Netdata в контейнере
Установите агента мониторинга на каждую ВМ и пробросьте метрики через SNMP или HTTP-интеграцию.

## ⚙️ Настройка

| Что заменить | Где | Пример |
|---|---|---|
| `sensor.pve_*` | весь файл | `sensor.proxmox_*` |
| `button.pve_*` | блок кнопок | `switch.pve_*` или `script.*` |
| `columns: 4` | верхний grid | `3` или `2` для меньших экранов |
| `hours_to_show: 24` | mini-graph-card | `12` для более детального графика |
| `height: 56` | mini-graph-card | `80` для крупных графиков |

## 🎭 Как это работает

### Склейка заголовка и панели кнопок

Заголовок (mushroom-template-card) и панель управления (horizontal-stack с button-card) выглядят как единая карточка за счёт:

- `border-radius: 8px 8px 0 0` у заголовка (скруглены только верхние углы)
- `border-radius: 0 0 8px 8px` у панели кнопок (скруглены только нижние углы)
- `border-bottom: none` у заголовка — убирает линию стыка
- Единая `border-left: 3px` фирменного цвета проходит через обе части
- Одинаковый `box-shadow` создаёт цельную тень

### Центрирование кнопок

Кнопки центрируются внутри панели через `card_mod`:

```css
#root {
  display: flex;
  justify-content: center;
  gap: 8px;
  padding: 8px 12px;
}
```

Это работает для `horizontal-stack` и даёт таскбар-стиль Windows 11.

## ❓ FAQ

**Q: Кнопки не по центру.**
A: Убедитесь, что card-mod установлен и включён. Некоторые темы переопределяют `#root` — добавьте `!important` в `display: flex`.

**Q: Шрифт Segoe UI не отображается.**
A: В Linux-среде (HAOS) Segoe UI не установлен по умолчанию, карточка fallback-ит на системный sans-serif. Визуальная разница минимальна. Для точного соответствия Win11 можно подключить веб-шрифт через `card-mod` на уровне темы.

**Q: На мобильном всё в столбик.**
A: Это штатное поведение `grid` с `columns: 4` — на узких экранах колонки складываются. Для фиксированной мобильной версии используйте `custom:layout-card` с `grid-media-query`.

**Q: Как добавить пятую ВМ?**
A: Скопируйте один из блоков `vertical-stack`, замените entity_id и фирменный цвет в трёх местах: `border-left`, цвет фона кнопок, цвет иконок.

**Q: Хочу попап вместо кнопок в шапке.**
A: Установите `browser_mod` (thomasloven), замените `tap_action` у заголовка на `action: fire-dom-event` → `browser_mod.popup` → в `content` вложите кнопки.

**Q: Кнопка Start для NAS не нужна — можно Wake-on-LAN?**
A: Добавьте ещё одну `button-card` с `icon: mdi:play` и `service: wake_on_lan.send_magic_packet` (интеграция wake_on_lan должна быть включена в configuration.yaml).

## 🛠 Troubleshooting

| Симптом | Решение |
|---|---|
| Графики пустые | Проверьте наличие сенсоров в Developer Tools → States |
| Кнопки не реагируют | Убедитесь, что `button.*` существуют и имеют `button.press` сервис |
| Белый текст на белом фоне | Замените `var(--ha-card-background)` на фиксированный цвет фона |
| Карточка ломается после обновления HA | Обновите card-mod и mushroom-cards до последней версии |
| Разрыв между заголовком и кнопками | Добавьте `margin: 0` к `ha-card` заголовка (убирает дефолтный отступ vertical-stack) |
| ZFS сенсоры недоступны | ProxmoxVE-интеграция не создаёт их автоматически — добавьте через REST или SNMP |

## 🗺 Roadmap

- [ ] Browser Mod: кнопки управления в попапе (чистый дашборд)
- [ ] Индикатор последнего бэкапа для каждой ВМ
- [ ] Wake-on-LAN для NAS и Q4OS
- [ ] Статус ZFS scrub / SMART для дисков
- [ ] Кастомные алерты при CPU > 90% более 5 минут
- [ ] Мобильная адаптация через layout-card с media-query

## 📄 Лицензия

MIT © [YOUR_NAME]. См. [LICENSE](LICENSE).

---

<p align="center"><i>Пригодилось — поставьте ⭐</i></p>
