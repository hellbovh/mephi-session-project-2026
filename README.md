# Сессионный проект по курсу "Безопасность GNU/Linux"

Выполнил: Петров Артем (GitHub: hellbovh)
Номер зачетной книжки: М265931

---

## Раздел 1. Установка дистрибутива

- **1.1. Настройка сети**: интерфейс `ens33` получает адрес 192.168.42.128/24
- **1.2. Настройка имени хоста**: `mephi-2026.domain.local` установлено через `hostnamectl set-hostname`, сохраняется после перезагрузки.
- **1.3. Проверка сетевой связности**: `ping -c 4 8.8.8.8`, результат сохранён в `ping.out`.

## Раздел 2. Управление программным обеспечением

- **2.1. Обновление дистрибутива**: выполнено `dnf update -y`, id8 в `dnf.out`.
- **2.2. Установка пакетов из репозиториев**: `dnf install nginx libcap-ng-utils`, id9
- **2.3. Установка локального RPM**: пакет `tcpdump-4.99.5-1.red80.x86_64.rpm` скачан через `dnf download` в `/tmp` и установлен через `rpm -Uvh --replacepkgs`. История транзакций сохранена в `dnf.out`.

## Раздел 3. Управление файловыми системами

- **3.1. Создание файловой системы**: на `/dev/sdb` создан раздел `/dev/sdb1`, отформатирован в `ext4` с меткой `MEPHI_WEB` через `mkfs.ext4 -L MEPHI_WEB`.
- **3.2. Монтирование**: создана точка `/mephi-web`. В `/etc/fstab`
  добавлена строка `LABEL=MEPHI_WEB /mephi-web ext4 defaults 0 2`
  Ручное монтирование выполнено командой `mount LABEL=MEPHI_WEB /mephi-web`. 
  Автомонтирование проверено через `umount` + `mount -a`.

## Раздел 4. Управление сервисами

- **4.1. Управление веб-сервером**: nginx запущен и добавлен в автозапуск командой `systemctl enable --now nginx`.
- **4.2. Журналирование**: вывод `journalctl -u nginx -b` сохранён в `journalctl.out`.

## Раздел 5. Управление доступом

- **5.1. DAC**: созданы группа `curators` (GID 4444) и группа `mephi-dev`;
  пользователи `user1` (UID 5501), `user2` (5502), `user3` (5503) входят
  в `mephi-dev`; кураторы `curator1`, `curator2` - в `curators`.
  Директория `/data/mephi-2026` имеет права `2770 root:mephi-dev`.
  SGID обеспечивает наследование группы; default ACL даёт группе
  `mephi-dev` права rw на новые файлы, `curators` - только чтение,
  остальным - ничего. Проверено через `sudo -u user1/user2/user3/curator1`.
- **5.2. Capabilities**: с `/usr/sbin/tcpdump` снят бит set-UID и выданы capabilities
  `cap_net_raw,cap_net_admin=eip` через `setcap`. Проверено запуском
  `sudo -u user1 tcpdump -c 5 -i ens33` - работает от обычного пользователя.
- **5.3. MAC (SELinux)**: SELinux в режиме `Enforcing`. Для `/mephi-web`
  задано правило контекста `semanage fcontext -a -t httpd_sys_content_t
  '/mephi-web(/.*)?'` и выполнено `restorecon -Rv /mephi-web`.
  В `/etc/nginx/nginx.conf` директива `root` заменена с
  `/usr/share/nginx/html` на `/mephi-web`. nginx корректно отдаёт
  страницы из новой директории.

## Раздел 6. Аутентификация

- **6.1. Ограничение входа**: `curator1` и `curator2` имеют shell
  `/sbin/nologin`, вход в систему им запрещён. Проверено через `su - curator1`
  (сообщение «This account is currently not available.»).
- **6.2. Управление паролями**: в `/etc/login.defs` установлено
  `PASS_MAX_DAYS 90`; для всех пользователей применено `chage -M 90`.
  В `/etc/security/pwquality.conf` установлено `minlen = 12`.
  Проверено попыткой смены пароля на короткий - отказ.

## Раздел 7. Тестирование

- **7.1. Web-страница**: создан `/mephi-web/index.html` с текстом
  `Hello from Student: M265931`.
- **7.2. Проверка доступности**: `curl http://localhost/` возвращает
  ожидаемый текст.

---

## Файлы

`mephi-screenshot.png` - скриншот с текстом "Hello from Student: M265931

`history.out` - история команд

`ping.out` - ответ `ping`

`dnf.out` - история менеджера пакетов

`stat.out` - права и ACL

`journalctl.out` - журнал nginx

`getcap.out` - tcpdump capabilities

`getenforce.out` - SELinux режим

`curl.out` - результат `curl` на веб-страницу

`fstab` - настройка монтирования

`passwd` - пользователи

`shadow` - хэши паролей

`group` - группы

`pwquality.conf` - политика паролей

`update.log`, `install.log` - логи обновлений и установок


