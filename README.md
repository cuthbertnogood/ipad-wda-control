# Управление iPad через WebDriverAgent

Инструкция по настройке внешнего управления iPad с помощью WebDriverAgent (WDA).

**Что можно делать:**
- Открывать любые приложения по Bundle ID
- Записывать экран
- Делать скриншоты, тапать, свайпать, вводить текст
- Управлять с Mac (по USB или Wi-Fi)

**Требования:**
- Mac с установленным Xcode
- iPad с включённым режимом разработчика
- Бесплатный Apple ID (профиль сгорает каждые 7 дней)

---

## 1. Подготовка iPad

1. На iPad откройте **Настройки → Конфиденциальность и безопасность → Режим разработчика** и включите его.
2. Перезагрузите iPad, если попросит.
3. Подключите iPad к Mac кабелем и нажмите **Доверять**.

---

## 2. Установка на Mac

Откройте Терминал и выполните:

```bash
# Homebrew (если ещё нет)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Инструменты для работы с устройством
brew install libimobiledevice

# Клонируем WebDriverAgent
git clone https://github.com/appium/WebDriverAgent.git
cd WebDriverAgent
open WebDriverAgent.xcodeproj
```

---

## 3. Подпись в Xcode

1. В Xcode откройте **Settings → Accounts** и добавьте свой Apple ID.
2. В проекте выберите таргет **WebDriverAgentRunner**.
3. Вкладка **Signing & Capabilities**:
   - Поставьте галочку **Automatically manage signing**.
   - В поле **Team** выберите свой **Personal Team**.
   - Измените **Bundle Identifier** на уникальный, например: `com.yourname.WebDriverAgentRunner`.

---

## 4. Запуск WDA

1. Выберите свой iPad в списке устройств сверху.
2. Нажмите **⌘ + U** (Product → Test).
3. Если на iPad появится сообщение о ненадёжном разработчике:
   - Откройте **Настройки → Основные → VPN и управление устройством**.
   - Найдите свой Apple ID и нажмите **Доверять**.
4. В консоли Xcode должна появиться строка:
   ```
   ServerURLHere->http://...:8100<-ServerURLHere
   ```

---

## 5. Проброс порта

В новом окне Терминала:

```bash
iproxy 8100 8100
```

Оставьте это окно открытым.

Проверка:

```bash
curl http://localhost:8100/status
```

Должен вернуться JSON.

---

## 6. Открытие приложений

Установите Python-клиент:

```bash
pip install facebook-wda
```

Пример скрипта:

```python
import wda

c = wda.Client('http://localhost:8100')

# Открыть Safari
c.session('com.apple.mobilesafari')

# Или открыть Настройки
c.app_launch('com.apple.Preferences')
```

Другие полезные Bundle ID:
- Safari: `com.apple.mobilesafari`
- Настройки: `com.apple.Preferences`
- Фото: `com.apple.mobileslideshow`
- Камера: `com.apple.camera`

---

## 7. Запись экрана

Самый простой способ — через Appium (использует MJPEG-поток WDA + ffmpeg).

Установите Appium:

```bash
npm install -g appium
appium driver install xcuitest
brew install ffmpeg
```

Пример на Python (через Appium Python Client):

```bash
pip install Appium-Python-Client
```

```python
from appium import webdriver
from appium.options.ios import XCUITestOptions
import time

options = XCUITestOptions()
options.platform_name = "iOS"
options.automation_name = "XCUITest"
options.udid = "ВАШ_UDID"  # получить через xcrun xctrace list devices или idevice_id -l
options.use_preinstalled_wda = True
options.updated_wda_bundle_id = "com.yourname.WebDriverAgentRunner"  # ваш Bundle ID

driver = webdriver.Remote("http://localhost:4723", options=options)

# Запуск записи
driver.start_recording_screen(timeLimit=60, videoFps=10)

# Ваши действия...
time.sleep(5)

# Остановка и сохранение
video = driver.stop_recording_screen()
with open("recording.mp4", "wb") as f:
    import base64
    f.write(base64.b64decode(video))

driver.quit()
```

---

## Важные замечания

- **Бесплатный Apple ID**: профиль сгорает каждые 7 дней. Просто снова нажмите **⌘ + U** в Xcode.
- iPad должен оставаться включённым, а сессия WDA — живой.
- Для работы по Wi-Fi (без кабеля) можно использовать `iproxy` с настройкой или Tailscale.
- Если WDA падает — снова запустите через Xcode (⌘ + U).

---

## Полезные ссылки

- [WebDriverAgent](https://github.com/appium/WebDriverAgent)
- [facebook-wda (Python)](https://github.com/openatx/facebook-wda)
- [Appium XCUITest Driver](https://appium.github.io/appium-xcuitest-driver/)

Удачи!