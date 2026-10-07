# Сессионный проект: безопасность GNU/Linux

РЕД ОС 8.0.3, хост `mephi-2026.domain.local`, зачётная книжка M265977.

Репозиторий: https://github.com/RokiRat/mephi-session-project-2026

Настройки выполнены в командной строке и сохраняются после перезагрузки: профиль NetworkManager, `hostnamectl`, `/etc/fstab`, `systemctl enable`, политики SELinux, capabilities, `/etc/login.defs` и `pwquality.conf`.

## Как выполнен каждый пункт

| Пункт | Что сделано | Чем подтверждается |
|---|---|---|
| 1.1 Сеть | Адрес интерфейса получается по DHCP | `ping.out` |
| 1.2 Имя хоста | Установлено `mephi-2026.domain.local` | `journalctl.out`, скриншот |
| 1.3 Связность | `ping -c 4 8.8.8.8`, 4 ответа, потерь нет | `ping.out` |
| 2.1 Обновление | `dnf -y update` | `dnf.out`, транзакция 2 |
| 2.2 Пакеты | Установлены `nginx`, `libcap-ng-utils`, `policycoreutils-python-utils` | `dnf.out`, транзакция 3 |
| 2.3 Локальный RPM | `tcpdump` скачан через `dnf download` в `/tmp` и установлен командой `rpm` | `getcap.out`, `history.out` |
| 3.1 Файловая система | На втором диске один раздел, `ext4`, метка `MEPHI_WEB` | `fstab`, `stat.out` |
| 3.2 Монтирование | Точка `/mephi-web`, автомонтирование по метке | `fstab`: `LABEL=MEPHI_WEB /mephi-web ext4 defaults 0 2` |
| 4.1 Веб-сервер | `nginx` запущен и включён в автозагрузку, корень сайта `/mephi-web` | `journalctl.out`, `curl.out` |
| 4.2 Журнал | Сообщения `nginx` текущей загрузки | `journalctl.out` |
| 5.1 DAC | Каталог `/data/mephi-2026`, режим `2770`, группа `developers`. Разработчики читают и пишут новые файлы, кураторы только читают, остальные доступа не имеют | `stat.out`, `passwd`, `group` |
| 5.2 Capabilities | У `/usr/sbin/tcpdump` снят setuid, выданы `cap_net_admin,cap_net_raw=ep` | `getcap.out` |
| 5.3 MAC | SELinux в режиме Enforcing, для `/mephi-web` постоянный контекст `httpd_sys_content_t` | `getenforce.out`, `stat.out` |
| 6.1 Вход | У `curator1` и `curator2` оболочка `/sbin/nologin`, локальный вход закрыт | `passwd` |
| 6.2 Пароли | Срок жизни пароля 90 дней, минимальная длина 12 символов | `shadow`, `pwquality.conf` |
| 7. Тест | В `/mephi-web/index.html` строка `Hello from Student: M265977`, страница отдаётся nginx | `curl.out`, `mephi-screenshot.png` |

## Учётные записи

| Пользователь | UID | Группа | Оболочка | Назначение |
|---|---|---|---|---|
| user1 | 5501 | developers (5500) | `/bin/bash` | разработчик |
| user2 | 5502 | developers (5500) | `/bin/bash` | разработчик |
| user3 | 5503 | developers (5500) | `/bin/bash` | разработчик |
| curator1 | 5504 | curators (4444) | `/sbin/nologin` | куратор, только чтение |
| curator2 | 5505 | curators (4444) | `/sbin/nologin` | куратор, только чтение |

Каталог проекта принадлежит `root:developers` и имеет setgid-бит (`2770`). Default ACL даёт разработчикам `rw`, кураторам `r` и закрывает доступ остальным.

## Артефакты

Файлы из `/etc` лежат в корне репозитория без каталога `etc`.

- `mephi-screenshot.png` — вывод `curl` со строкой студента
- `history.out` — история команд
- `ping.out` — проверка сети
- `dnf.out` — история транзакций
- `stat.out` — права и контекст SELinux
- `journalctl.out` — запуск nginx
- `getcap.out` — capabilities `tcpdump`
- `getenforce.out` — режим SELinux
- `curl.out` — ответ веб-сервера
- `fstab` — монтирование `/mephi-web`
- `passwd`, `shadow`, `group` — пользователи, пароли и группы
- `pwquality.conf` — минимальная длина пароля
