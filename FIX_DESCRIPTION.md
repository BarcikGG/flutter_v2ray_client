# Исправление проблемы с setUpListener

## Проблема

При каждом вызове `startV2Ray()` создается новый экземпляр `V2rayVPNService`, который в методе `onCreate()` вызывает `setUpListener(this)`. Это приводит к:

1. Переинициализации `coreController` - теряется связь с callback'ами
2. Перезаписи `v2rayServicesListener` - теряется предыдущий listener
3. Ошибке в логах: `"setUpListener => new initialize from V2rayVPNService"`
4. Callback `onStatusChanged` не вызывается, так как `coreController` пересоздается

## Решение

Добавлена проверка в метод `setUpListener()`:
- Если `coreController` уже инициализирован и `v2rayServicesListener` установлен
- Просто обновляем `v2rayServicesListener` без переинициализации `coreController`
- Это сохраняет callback'и и позволяет статусу обновляться правильно

## Изменения

**Файл:** `android/src/main/java/dev/amirzr/flutter_v2ray_client/v2ray/core/V2rayCoreManager.java`

**Метод:** `setUpListener(Service targetService)`

**Добавлено:**
```java
// ИСПРАВЛЕНИЕ: Если listener уже установлен и core инициализирован,
// просто обновляем listener без переинициализации
if (isLibV2rayCoreInitialized && v2rayServicesListener != null && coreController != null) {
    Log.d(V2rayCoreManager.class.getSimpleName(), "setUpListener => updating listener for existing core from "
            + targetService.getClass().getSimpleName());
    v2rayServicesListener = (V2rayServicesListener) targetService;
    return;
}
```

**Также изменено:**
- Уровень логирования с `Log.e` на `Log.d` для сообщения о новой инициализации (строка 184)

## Ожидаемый результат

После исправления:
- ✅ Callback `onStatusChanged` будет вызываться правильно
- ✅ Статус VPN будет обновляться (connecting → connected)
- ✅ Трафик будет передаваться через callback
- ✅ Не будет ошибки "setUpListener => new initialize"
- ✅ VPN будет подключаться успешно

