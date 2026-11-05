[README.md](https://github.com/user-attachments/files/34444910/README.md)[README.md](https://github.com/user-attachments/files/34444908/README.md)
# 📎 Paperwork Inc.

🇷🇺 Русский · [🇬🇧 English](#english)

> *Approve documents. Open departments. Approve more documents. Per my last memo.*

Сатирическая idle-игра про бюрократию для Telegram Mini App. Игрок управляет корпорацией Paperwork Inc. Корпорация одобряет документы, на них открываются новые отделы, а те одобряют ещё больше документов.

Вся игра умещается в один `index.html`: без сборки, фреймворков и картинок. Единственная внешняя зависимость — SDK Telegram.

🤖 Игра сделана с помощью **Claude Opus 5.5** от Anthropic: код, баланс и тексты.

**▶️ Играть:** `https://t.me/<бот>/<short_name>`
**🌐 В браузере:** `https://<логин>.github.io/<репозиторий>/`

---

## Как играть

Это idle, а не кликер. Доход идёт сам, в том числе пока игра закрыта.

1. В начале нажмите **Sign manually**, чтобы набрать на первый отдел.
2. Покупайте **отделы**. Каждый приносит документы в секунду.
3. Покупайте **апгрейды**: множители, скидки и улучшения офлайн-дохода.
4. Закройте игру и вернитесь через несколько часов. Отчёт покажет, сколько компания одобрила без вас.
5. Через 2–3 дня станет доступна **реструктуризация**. Она обнуляет прогресс и даёт Seniority, а с ним постоянный бонус к доходу.

### Ресурсы

| Ресурс | Что это |
|---|---|
| 📄 **Documents** | Основная валюта, за неё покупаются отделы и большинство апгрейдов |
| 🔏 **Stamps** | Вторая валюта. Открывается апгрейдом Official Rubber Stamp, нужна для продвинутых улучшений |
| 🎖️ **Seniority** | Престиж-валюта. Каждая единица даёт +10% ко всему доходу навсегда |

### Механики

- **10 отделов**, от Intern's Desk до Head Office of Head Offices. Цена растёт как `база × 1.15^количество`.
- **Вехи:** каждые 25 штук одного отдела удваивают его выработку.
- **25 апгрейдов.** Видны только почти доступные и уже купленные.
- **Офлайн-доход:** на старте начисляется 50% дохода и не больше чем за 8 часов. Апгрейды поднимают это до 100% и 20 часов.
- **Restructuring:** Seniority = `floor(√(всего заработано / 5·10⁹))`. Нужно заработать минимум 300B за период.

---

## Деплой

### 1. GitHub Pages

1. Положите `index.html` в корень репозитория.
2. Откройте **Settings → Pages → Build and deployment** и выберите Source: *Deploy from a branch*, ветку `main`, папку `/ (root)`.
3. Через 1–2 минуты игра будет доступна по адресу `https://<логин>.github.io/<репозиторий>/`.

### 2. Telegram-бот

1. В [@BotFather](https://t.me/BotFather) отправьте `/newbot` и создайте бота.
2. Отправьте `/newapp`, выберите бота, укажите название, описание и картинку 640×360. В поле Web App URL вставьте ссылку на GitHub Pages и придумайте short name.
3. Игра будет открываться по ссылке `https://t.me/<бот>/<short_name>`.
4. Если нужна кнопка в чате с ботом, настройте её так: `/mybots` → бот → **Bot Settings → Menu Button** → тот же URL.

После каждого пуша в `main` GitHub Pages обновляется сам, перенастраивать бота не нужно.

---

## Разработка

### Локальный запуск

Откройте `index.html` в браузере. Для мобильного вида включите эмуляцию телефона в DevTools.

Если нужен локальный сервер:

```bash
python3 -m http.server 8000
# → http://localhost:8000
```

Вне Telegram игра работает так же, только:
- сохранение идёт в `localStorage` вместо CloudStorage;
- виброотклика нет;
- цвета берутся из встроенной «бумажной» палитры, светлой или тёмной по настройкам системы.

### Читы для теста

Состояние игры лежит в глобальной переменной `S`. В консоли браузера можно выполнить:

```js
S.docs = 1e9; render();                                  // накинуть документов
S.lastSeen = Date.now() - 5 * 3600e3; save(); location.reload(); // имитировать 5 часов офлайна
S.runEarned = S.allTimeEarned = 4e11; render();          // открыть реструктуризацию
```

Полностью сбросить прогресс можно кнопкой **Shred all records** во вкладке Restructuring.

### Где что править

Весь баланс и тексты собраны в начале `<script>`:

| Объект | Что внутри |
|---|---|
| `CONFIG` | Рост цен, офлайн-лимит и эффективность, вехи, пороги открытия вкладок, формула престижа |
| `DEPARTMENTS` | Отделы: цена, выработка, эмодзи, номер формы, описание |
| `UPGRADES` | Апгрейды: цена, валюта, условия появления, эффект, описание |
| `TEXTS` | Все строки интерфейса, шутки для окна возвращения, служебные записки |

Код между маркерами `// ===== ENGINE START =====` и `// ===== ENGINE END =====` — это чистая логика без DOM. Её можно загрузить в Node и прогнать симуляцию баланса.

### Типы эффектов апгрейдов

| `effect.type` | Параметры | Что делает |
|---|---|---|
| `dept` | `dept`, `mult` | Множитель выработки одного отдела |
| `global` | `mult` | Множитель всей выработки |
| `cost` | `mult` | Множитель цен отделов (например, `0.9` даёт −10%) |
| `offlineCap` | `hours` | Увеличивает лимит офлайна |
| `offlineEff` | `add` | Повышает эффективность офлайна (максимум 100%) |
| `unlockStamps` | — | Открывает печати |
| `stampMult` | `mult` | Множитель производства печатей |
| `seniority` | `add` | Увеличивает бонус за единицу Seniority |

Условия появления (`req`) могут включать `dept` + `count`, `upgrade`, `earned` (заработано за период) и `restructures`.

### Сохранение

- Ключ хранилища: `paperwork_inc_save_v1`.
- Игра сохраняется каждые 10 секунд, а также при сворачивании (`visibilitychange`) и закрытии (`pagehide`).
- Внутри Telegram запись идёт и в CloudStorage, и в `localStorage`. При загрузке берётся более свежая версия.
- Сохранение занимает около 500 байт при лимите CloudStorage в 4096 байт на ключ.

Если поменяется формат сохранения, обновите функцию `migrate()` или смените ключ.

---

<a id="english"></a>

# 🇬🇧 Paperwork Inc. — English

> *Approve documents. Open departments. Approve more documents. Per my last memo.*

A satirical idle game about bureaucracy, built as a Telegram Mini App. You run Paperwork Inc., a corporation that approves documents in order to open new departments, which approve even more documents.

The whole game is a single `index.html`: no build step, no frameworks, no images. The only external dependency is the Telegram SDK.

🤖 Made with **Claude Opus 5.5** by Anthropic: code, balancing and writing.

**▶️ Play:** `https://t.me/<bot>/<short_name>`
**🌐 In a browser:** `https://<username>.github.io/<repository>/`

## How to play

It's an idle game, not a clicker. Income accrues on its own, even while the game is closed.

1. Tap **Sign manually** at the start to afford your first department.
2. Buy **departments**. Each one produces documents per second.
3. Buy **upgrades**: multipliers, discounts and offline improvements.
4. Close the game and come back a few hours later. A report shows what the company approved without you.
5. After 2–3 days, **Restructuring** unlocks. It resets your progress and grants Seniority, a permanent income bonus.

### Resources

| Resource | What it is |
|---|---|
| 📄 **Documents** | Main currency for departments and most upgrades |
| 🔏 **Stamps** | Second currency. Unlocked by the Official Rubber Stamp upgrade, spent on advanced upgrades |
| 🎖️ **Seniority** | Prestige currency. Each point permanently adds +10% to all income |

### Mechanics

- **10 departments**, from Intern's Desk to Head Office of Head Offices. Price grows as `base × 1.15^owned`.
- **Milestones:** every 25 units of a department double its output.
- **25 upgrades.** Only those that are nearly affordable or already bought are shown.
- **Offline income:** starts at 50% efficiency for up to 8 hours. Upgrades raise this to 100% and 20 hours.
- **Restructuring:** Seniority = `floor(√(total earned / 5·10⁹))`. Requires at least 300B earned in the current period.

## Deployment

### 1. GitHub Pages

1. Put `index.html` in the repository root.
2. Go to **Settings → Pages → Build and deployment** and choose Source: *Deploy from a branch*, branch `main`, folder `/ (root)`.
3. In 1–2 minutes the game is live at `https://<username>.github.io/<repository>/`.

### 2. Telegram bot

1. In [@BotFather](https://t.me/BotFather), send `/newbot` and create a bot.
2. Send `/newapp`, pick the bot, set a title, description and a 640×360 image. Paste the GitHub Pages link as the Web App URL and choose a short name.
3. The game opens at `https://t.me/<bot>/<short_name>`.
4. For a button in the bot chat: `/mybots` → your bot → **Bot Settings → Menu Button** → the same URL.

GitHub Pages redeploys on every push to `main`; the bot needs no changes.

## Development

### Running locally

Open `index.html` in a browser. Use your browser's device emulation for the mobile layout.

If you need a local server:

```bash
python3 -m http.server 8000
# → http://localhost:8000
```

Outside Telegram the game works the same, except:
- saves go to `localStorage` instead of CloudStorage;
- there is no haptic feedback;
- colors come from the built-in paper palette, light or dark depending on system settings.

### Test cheats

The game state lives in the global variable `S`. In the browser console:

```js
S.docs = 1e9; render();                                  // add documents
S.lastSeen = Date.now() - 5 * 3600e3; save(); location.reload(); // simulate 5 hours offline
S.runEarned = S.allTimeEarned = 4e11; render();          // unlock restructuring
```

To wipe all progress, use **Shred all records** on the Restructuring tab.

### Where to tweak things

All balance values and texts sit at the top of the `<script>`:

| Object | Contents |
|---|---|
| `CONFIG` | Cost growth, offline cap and efficiency, milestones, tab unlock thresholds, prestige formula |
| `DEPARTMENTS` | Departments: cost, output, emoji, form number, description |
| `UPGRADES` | Upgrades: cost, currency, unlock conditions, effect, description |
| `TEXTS` | All UI strings, return-screen jokes, office memos |

Code between `// ===== ENGINE START =====` and `// ===== ENGINE END =====` is pure logic with no DOM. You can load it in Node to run balance simulations.

### Upgrade effect types

| `effect.type` | Parameters | Effect |
|---|---|---|
| `dept` | `dept`, `mult` | Multiplies one department's output |
| `global` | `mult` | Multiplies all output |
| `cost` | `mult` | Multiplies department prices (e.g. `0.9` = −10%) |
| `offlineCap` | `hours` | Raises the offline cap |
| `offlineEff` | `add` | Raises offline efficiency (capped at 100%) |
| `unlockStamps` | — | Unlocks stamps |
| `stampMult` | `mult` | Multiplies stamp production |
| `seniority` | `add` | Raises the bonus per Seniority point |

Unlock conditions (`req`) can include `dept` + `count`, `upgrade`, `earned` (earned this period) and `restructures`.

### Saving

- Storage key: `paperwork_inc_save_v1`.
- The game saves every 10 seconds, on minimize (`visibilitychange`) and on close (`pagehide`).
- Inside Telegram it writes to both CloudStorage and `localStorage`, and loads whichever is newer.
- A save is about 500 bytes, well under CloudStorage's 4096-byte limit per key.

If you change the save format, update `migrate()` or bump the key.
