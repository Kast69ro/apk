# ActivBank QR — POS-приложение для Sunmi

Стек: **Capacitor 6 → Android WebView** + один самодостаточный HTML-файл
(`apk/www/index.html`; React-рантайм, QR-энкодер и логотип вшиты внутрь, работает офлайн).
Нативного кода нет, Android Studio не нужна — папку `android` генерирует CI.

Бэкенд: `http://10.64.20.101:8888` — `POST /pos/login`, `GET /pos/terminal`, `GET /pos/history`.
Если хост недоступен, приложение показывает демо-данные того же формата.

## 1. Что положить в репозиторий

    apk/www/index.html            приложение (перезаливать при изменении дизайна)
    apk/package.json
    apk/capacitor.config.json
    assets/activbank-qr-app.png   логотип -> иконка приложения
    .github/workflows/build-apk.yml

## 2. Ключ подписи (нужен, чтобы Sunmi приняли APK)

На макбуке (нужна только Java):

    keytool -genkeypair -v -keystore release.keystore -alias activbank \
      -keyalg RSA -keysize 2048 -validity 10000
    base64 -i release.keystore | pbcopy

В GitHub → Settings → Secrets and variables → Actions → New repository secret:

    KEYSTORE_B64        (вставить из буфера)
    KEYSTORE_PASSWORD
    KEY_ALIAS           activbank
    KEY_PASSWORD

Ключ и пароли сохрани: все будущие обновления APK должны быть подписаны им же.

## 3. Получить APK из GitHub

1. Push в `main` — либо вкладка **Actions → Build APK → Run workflow** (кнопка справа).
2. Дождись зелёной галочки (~5–8 мин, первый запуск дольше: качается Android SDK).
3. Открой запуск → внизу блок **Artifacts** → скачай `activbank-qr-apk.zip`.
4. Внутри два файла:
   * `app-debug.apk` — для проверки на своём терминале (`adb install -r app-debug.apk`);
   * `app-release.apk` — подписанный, **его отправляй саппорту Sunmi**.

Если сборка упала — открой упавший шаг в логе; чаще всего это отсутствующий
`assets/activbank-qr-app.png` или опечатка в секретах.

## 4. Обновить приложение

Пересобери `index.html` из дизайна, положи в `apk/www/index.html`, подними
`versionCode`/`versionName` в `android/app/build.gradle` (или просто в новом теге),
push → новый артефакт.

## 5. Sunmi-нюансы

* Манифест патчится в CI: `usesCleartextTraffic="true"` (бэкенд по http),
  `keepScreenOn="true"`, портретная ориентация.
* minSdk 22+ — покрывает P2 / P2 Pro / V2 на Android 7–11.
* Киоск-режим (запрет выхода из приложения) настраивается через Sunmi MDM,
  пересборка не нужна.
