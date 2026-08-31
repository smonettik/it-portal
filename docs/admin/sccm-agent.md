# Агент SCCM (Configuration Manager)

Инструкция по установке клиентского агента SCCM (Configuration Manager) на Windows-устройства.

## Предварительные требования

- Права локального администратора на целевом устройстве;
- устройство в домене Active Directory и имеет доступ к серверу SCCM;
- открытые порты 80/443 (HTTP/HTTPS) для связи с сервером;
- .NET Framework 4.5+ и Windows PowerShell 3.0+.

## Способы установки

### Способ 1 — через консоль SCCM (рекомендуется)

1. Откройте консоль Configuration Manager.
2. Перейдите в «Активы и соответствие» → «Устройства».
3. Выберите коллекцию устройств для установки.
4. Правая кнопка мыши → «Установить клиент», укажите параметры и расписание.

### Способ 2 — вручную (ccmsetup.exe)

    \\SCCM_Server\SMS_SiteCode\Client\ccmsetup.exe

    ccmsetup.exe /mp:sccm01.contoso.com SMSSITECODE=ABC

### Способ 3 — через групповую политику (GPO)

1. Создайте новый объект групповой политики.
2. Перейдите: Конфигурация компьютера → Политики → Конфигурация Windows → Сценарии (запуск/завершение).
3. Добавьте сценарий запуска с командой `ccmsetup.exe`.
4. Назначьте GPO нужным подразделениям.

### Способ 4 — сценарий входа (Logon Script)

Пример VBScript:

    Set objShell = CreateObject("WScript.Shell")
    objShell.Run "\\sccm01\SMS_ABC\Client\ccmsetup.exe /mp:sccm01.contoso.com", 0, True

## Параметры ccmsetup.exe

- **/mp** — указывает точку управления, пример: `/mp:sccm01.contoso.com`
- **SMSSITECODE** — код сайта SCCM, пример: `SMSSITECODE=ABC`
- **CCMINSTALLDIR** — путь установки
- **/logon** — установить при следующем входе в систему
- **/forceinstall** — принудительная установка
- **/noservice** — не устанавливать службу

## Проверка установки

Службы (должны быть в состоянии Running):

    Get-Service -Name CcmExec, smstsmgr, ccmsetup

Папки и журналы:

- `C:\Windows\CCM` — основная папка клиента;
- `C:\Windows\CCMSetup` — папка установки;
- `C:\Windows\CCM\Logs` — журналы;
- `ccmsetup.log` — процесс установки;
- `client.msi.log` — установка MSI-пакета;
- `CcmExec.log` — работа основной службы;
- `LocationServices.log` — обнаружение сайта.

## Команды PowerShell для управления

    Get-WmiObject -Namespace root\ccm -Class SMS_Client
    Restart-Service -Name CcmExec -Force
    Invoke-WmiMethod -Namespace root\ccm -Class SMS_Client -Name ResetPolicy -ArgumentList 1
    Invoke-WmiMethod -Namespace root\ccm -Class SMS_Client -Name RefreshManagementPoints

## Устранение неполадок

1. **Ошибка 0x80004005** — проверьте права доступа к сетевой папке и что брандмауэр не блокирует доступ.
2. **Клиент не появляется в консоли** — проверьте `C:\Windows\CCM\Logs\ClientIDManagerStartup.log` и убедитесь, что устройство в правильной границе (Boundary).
3. **Служба не запускается** — переустановите клиент:

    \\sccm01\SMS_ABC\Client\ccmsetup.exe /uninstall
    \\sccm01\SMS_ABC\Client\ccmsetup.exe /mp:sccm01.contoso.com SMSSITECODE=ABC

!!! tip "Рекомендации"
    Тестируйте установку на контрольной группе устройств, используйте коллекции для контролируемого развёртывания, отслеживайте установки через отчёты SCCM и регулярно обновляйте версию клиента.