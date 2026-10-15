# Конфигурация виртуальной машины devops-vm

## 1. Параметры машины

| Параметр | Значение |
| :--- | :--- |
| Наименование | devops-vm |
| Тип и версия гостевой системы | Linux / Ubuntu (64-bit), Ubuntu Server 24.04 LTS |
| Объём оперативной памяти | 2048 МБ |
| Количество ядер процессора | 2 |
| Объём дискового накопителя | 25 ГБ, динамически расширяемый |
| Гипервизор | Oracle VirtualBox 7.x |

## 2. Сетевые интерфейсы

| Адаптер | Тип | Интерфейс | Адрес | Назначение |
| :--- | :--- | :--- | :--- | :--- |
| Адаптер 1 | NAT | enp0s3 | 10.0.2.15/24 | Доступ в Интернет, проброс портов |
| Адаптер 2 | Host-only | enp0s8 | 192.168.56.101/24 | Прямой доступ с хостовой системы |

## 3. Правило проброса портов

| Поле | Значение |
| :--- | :--- |
| Имя | ssh |
| Протокол | TCP |
| IP хоста | 127.0.0.1 |
| Порт хоста | 2222 |
| IP гостя | 10.0.2.15 |
| Порт гостя | 2222 |

Настраивается: VirtualBox → Настройки → Сеть → Адаптер 1 (NAT) → Дополнительно → Проброс портов.

## 4. Учётные записи

| Имя | UID | Группы | Способ аутентификации |
| :--- | :--- | :--- | :--- |
| student | 1000 | student, sudo | пароль (только для sudo) |
| devops | 1001 | devops, sudo | SSH-ключ ed25519 |

Ключ `devops` сгенерирован на хостовой системе:
Открытый ключ размещён в `/home/devops/.ssh/authorized_keys`, права: `.ssh` — 700, `authorized_keys` — 600.

## 5. Служба SSH

| Параметр | Значение |
| :--- | :--- |
| Порт | 2222 |
| Файл конфигурации | /etc/ssh/sshd_config.d/99-hardening.conf |
| PermitRootLogin | no |
| PasswordAuthentication | no |
| PubkeyAuthentication | yes |
| PermitEmptyPasswords | no |
| MaxAuthTries | 3 |
| LoginGraceTime | 30 |
| AllowUsers | devops |
| X11Forwarding | no |
| ClientAliveInterval | 300 |
| ClientAliveCountMax | 2 |

Дополнительно:
- В `/etc/ssh/sshd_config.d/50-cloud-init.conf` строка `PasswordAuthentication yes` закомментирована (`#PasswordAuthentication yes`)
- Socket-активация отключена: `sudo systemctl disable --now ssh.socket`
- Сервис включён в автозапуск: `sudo systemctl enable --now ssh.service`

## 6. Правила межсетевого экрана (UFW)

Политики по умолчанию:
- `deny (incoming)`
- `allow (outgoing)`
- `disabled (routed)`

Правила:

| Порт | Действие | Комментарий |
| :--- | :--- | :--- |
| 2222/tcp | LIMIT IN | SSH rate-limited |
| 80/tcp | ALLOW IN | HTTP |
| 443/tcp | ALLOW IN | HTTPS |

Логирование: `medium`.

Команды для воспроизведения:
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 80/tcp comment 'HTTP'
sudo ufw allow 443/tcp comment 'HTTPS'
sudo ufw limit 2222/tcp comment 'SSH rate-limited'
sudo ufw logging medium
sudo ufw enable