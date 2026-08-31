# Агент GLPI (инвентаризация)

GLPI-агент (FusionInventory Agent) — программа для сбора информации об оборудовании и программном обеспечении компьютеров в сети. Агент отправляет данные инвентаризации на сервер GLPI для автоматического учёта IT-активов.

## Подготовка

Перед установкой убедитесь, что есть:

- адрес сервера GLPI (например, `https://glpi.example.com`);
- токен аутентификации (если настроен на сервере);
- права администратора на целевом компьютере.

Определите ОС: Windows (7/8/10/11, Server 2012+), Linux (Debian, Ubuntu, RHEL, CentOS, Fedora) или macOS.

## Установка на Windows

**Способ 1 — установщик MSI**

1. Скачайте `fusioninventory-agent-windows-*.msi` из официального репозитория GitHub FusionInventory.
2. Запустите установщик и примите лицензионное соглашение.
3. Выберите компоненты (обычно по умолчанию) и путь установки.
4. Укажите адрес сервера GLPI и интервал инвентаризации (по умолчанию 1 час).
5. Завершите установку — агент встанет как служба Windows и запустится автоматически.

**Способ 2 — портативная версия**

1. Скачайте портативную версию (ZIP) и распакуйте в нужную папку.
2. Настройте файл `agent.cfg`.
3. Запустите агент вручную или создайте задание в Планировщике задач.

## Установка на Linux

Debian / Ubuntu:

    sudo apt-get update && sudo apt-get install -y wget
    wget -O - https://debian.fusioninventory.org/debian/archive.key | sudo apt-key add -
    echo "deb https://debian.fusioninventory.org/debian/ stable main" | sudo tee /etc/apt/sources.list
    sudo apt-get update && sudo apt-get install -y fusioninventory-agent
    sudo nano /etc/fusioninventory/agent.cfg

RHEL / CentOS / Fedora:

    sudo yum install -y epel-release
    sudo yum install -y wget
    sudo yum install -y fusioninventory-agent
    sudo nano /etc/fusioninventory/agent.cfg

## Настройка agent.cfg

    server = https://ваш-сервер-glpi/plugins/fusioninventory
    delaytime = 3600
    no-ssl-check = 0
    logfile = /var/log/fusioninventory/agent.log
    tasks = inventory

## Команды управления

Linux / macOS:

    sudo systemctl status fusioninventory-agent
    sudo systemctl start fusioninventory-agent
    sudo systemctl stop fusioninventory-agent
    sudo fusioninventory-agent --force
    sudo fusioninventory-agent --debug --test

Windows (командная строка администратора):

    sc query fusioninventory-agent
    net start fusioninventory-agent
    net stop fusioninventory-agent

## Проверка

1. Проверьте лог: Linux — `/var/log/fusioninventory/agent.log`, Windows — `C:\Program Files\FusionInventory-Agent\agent.log`.
2. Убедитесь, что появилось сообщение `Result sent successfully to server`.
3. На сервере GLPI: «Активы» → «Компьютеры» — новый компьютер должен появиться в списке.

## Расширенные возможности

- **Автоматическое развёртывание:** `tasks = inventory, deploy`
- **Сканирование сети:** `tasks = netdiscovery, netinventory`
- **Прокси-сервер:** параметры `proxy`, `proxy-user`, `proxy-pwd`

## Устранение неполадок

1. **Агент не отправляет данные** — проверьте подключение к серверу (`ping`), настройки брандмауэра, увеличьте уровень лога (`loglevel = 3`).
2. **Ошибки SSL** — для теста временно отключите проверку: `no-ssl-check = 1`.

!!! warning "Важно"
    Не отключайте проверку SSL в рабочей среде. Убедитесь, что версия агента совместима с сервером GLPI. Для массовой установки используйте GPO, Ansible или Puppet.