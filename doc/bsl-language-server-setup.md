# Настройка BSL Language Server в Cursor / VS Code

Инструкция для ручной установки **BSL Language Server** и подключения к расширению **Language 1C (BSL)** (`1c-syntax.language-1c-bsl`), когда автозагрузка с GitHub не работает (ошибка TLS, прокси, блокировка сети).

## Что понадобится

| Компонент | Описание |
|-----------|----------|
| Cursor или VS Code | Редактор с поддержкой расширений |
| Расширение **Language 1C (BSL)** | Издатель: `1c-syntax` |
| BSL Language Server | Скачивается с GitHub (см. ниже) |
| Java | **Не нужна**, если используете Windows-сборку с `bsl-language-server.exe` (в комплекте есть встроенная JRE) |

## 1. Установить расширение

1. Откройте панель расширений (`Ctrl+Shift+X`).
2. Найдите **Language 1C (BSL)** (издатель **1c-syntax**).
3. Установите и перезагрузите окно редактора при запросе.

## 2. Скачать BSL Language Server

1. Откройте в браузере: https://github.com/1c-syntax/bsl-language-server/releases/latest
2. Скачайте архив для Windows, например **`bsl-language-server_win.zip`**.
3. Распакуйте в постоянный каталог, например:

   ```text
   C:\Tools\bsl-language-server\
   ```

### Ожидаемая структура каталога (Windows-сборка jpackage)

```text
C:\Tools\bsl-language-server\
├── bsl-language-server.exe          ← основной исполняемый файл (указывать в настройках)
├── app\
│   └── bsl-language-server-0.29.0-exec.jar
└── runtime\                           ← встроенная JRE, отдельно настраивать не нужно
```

### Проверка установки

В PowerShell:

```powershell
& "C:\Tools\bsl-language-server\bsl-language-server.exe" --version
```

Ожидаемый вывод содержит строку вида `version: 0.29.0`. Предупреждения `sun.misc.Unsafe` — нормальны, на работу не влияют.

## 3. Настроить расширение

Есть два способа: через UI или через `settings.json`.

### Способ A — через интерфейс настроек

1. `Ctrl+,` → в поиске введите `@ext:1c-syntax.language-1c-bsl`.
2. Найдите и измените параметры:

| Параметр | Значение |
|----------|----------|
| **Download Language Server** | **выключить** (снять галочку) |
| **Language Server External Jar** | **включить** |
| **Language Server External Jar Path** | `C:\Tools\bsl-language-server\bsl-language-server.exe` |
| **Language Server Enabled** | включить |

> Параметр называется «External Jar», но на Windows указывается путь к **`bsl-language-server.exe`**.

### Способ B — через settings.json (рекомендуется)

`Ctrl+Shift+P` → **Preferences: Open User Settings (JSON)** и добавьте:

```json
{
  "language-1c-bsl.downloadLanguageServer": false,
  "language-1c-bsl.languageServerExternalJar": true,
  "language-1c-bsl.languageServerExternalJarPath": "C:\\Tools\\bsl-language-server\\bsl-language-server.exe",
  "language-1c-bsl.languageServerEnabled": true,
  "language-1c-bsl.languageAutocomplete": "ru"
}
```

Замените путь, если распаковали сервер в другой каталог. В JSON обратные слэши удваиваются: `\\`.

### Альтернатива: только JAR + системная Java

Если используете **`bsl-language-server.jar`** (не Windows exe-сборку):

```json
{
  "language-1c-bsl.downloadLanguageServer": false,
  "language-1c-bsl.languageServerExternalJar": true,
  "language-1c-bsl.languageServerExternalJarPath": "C:\\Tools\\bsl-language-server\\bsl-language-server.jar",
  "language-1c-bsl.languageServerExternalJarJavaPath": "C:\\Program Files\\Java\\jdk-17\\bin\\java.exe"
}
```

## 4. Перезапустить language server

1. `Ctrl+Shift+P`
2. Команда: **Language 1C (BSL): Restart the BSL Language Server**
3. Откройте любой файл `.bsl` в проекте — должны появиться подсказки и диагностика.

## 5. Проверка, что всё работает

- В статусной строке нет ошибки «Could not update/download BSL Language Server».
- В `.bsl`-файле работает автодополнение (`Ctrl+Space`).
- Проблемы кода подсвечиваются (если в проекте есть `.bsl-language-server.json`).

### Конфигурация анализа в проекте (опционально)

В корне workspace можно положить файл **`.bsl-language-server.json`** — правила диагностики, язык, пути к исходникам. Расширение подхватывает его автоматически (параметр **Language Server Configuration**, по умолчанию `.bsl-language-server.json`).

## 6. Обновление сервера

1. Скачайте новый релиз с https://github.com/1c-syntax/bsl-language-server/releases
2. Распакуйте поверх старого каталога или в новый и обновите **Language Server External Jar Path** при смене пути.
3. Перезапустите language server (шаг 4).

Автообновление через расширение можно включить обратно (`downloadLanguageServer: true`), если сеть до GitHub стабильна.

## Устранение неполадок

### Ошибка: «Could not fetch from GitHub releases API» / TLS

**Причина:** Cursor не может скачать сервер с GitHub (прокси, VPN, антivirus, нестабильная сеть).

**Решение:** ручная установка по этой инструкции + `downloadLanguageServer: false`.

### Ошибка: «API rate limit exceeded»

Создайте токен GitHub (без дополнительных scope): https://github.com/settings/tokens

В настройках:

```json
"language-1c-bsl.githubToken": "ghp_xxxxxxxx"
```

Или переменная окружения `LANGUAGE_1C_BSL_GITHUB_TOKEN`.

### Корпоративный прокси

В `settings.json`:

```json
{
  "http.proxy": "http://user:password@proxy.company:8080",
  "http.proxyStrictSSL": false
}
```

`proxyStrictSSL: false` — только если прокси подменяет SSL-сертификаты.

### Нужна только подсветка синтаксиса, без анализа

```json
"language-1c-bsl.languageServerEnabled": false
```

### Логи language server

```json
"bsl.trace.server": "verbose"
```

Лог: **View → Output** → в выпадающем списке выберите **BSL Language Server**.

## Справочник параметров расширения

| Ключ | Назначение |
|------|------------|
| `language-1c-bsl.downloadLanguageServer` | Автоскачивание с GitHub (`false` при ручной установке) |
| `language-1c-bsl.languageServerExternalJar` | Использовать локальный сервер |
| `language-1c-bsl.languageServerExternalJarPath` | Путь к `bsl-language-server.exe` или `.jar` |
| `language-1c-bsl.languageServerExternalJarJavaPath` | Путь к `java.exe` (только для `.jar`) |
| `language-1c-bsl.languageServerBaseInstallDir` | Каталог автоустановки (не для ручного exe) |
| `language-1c-bsl.languageServerEnabled` | Включить/выключить language server |
| `language-1c-bsl.languageServerConfiguration` | Путь к `.bsl-language-server.json` |
| `language-1c-bsl.githubToken` | Токен GitHub при лимите API |

## Текущая конфигурация в этом проекте

На машине разработчика сервер установлен в:

```text
C:\Tools\bsl-language-server\bsl-language-server.exe
```

Версия: **0.29.0** (проверено командой `--version`).

Настройки Cursor (User settings) уже содержат блок для локального сервера; при переносе на другой ПК повторите шаги 2–4 с актуальным путём.
