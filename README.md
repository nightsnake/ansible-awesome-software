# Ansible Role: Awesome Software Pack

[![Ansible Role](https://img.shields.io/badge/ansible-role-blue.svg)](https://galaxy.ansible.com/nightsnake/ansible-awesome-software)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

Ansible role для установки репозиториев и программного обеспечения на Debian/Ubuntu/Linux Mint системы. Роль оптимизирована для автоматизации настройки рабочей станции с поддержкой формата deb822 и групповой установки пакетов.

## Поддерживаемые дистрибутивы

| Дистрибутив | Версии | Кодовые имена |
|-------------|--------|---------------|
| **Ubuntu** | 22.04, 24.04, 26.04 | Jammy, Noble, Wily |
| **Debian** | 12, 13 | Bookworm, Trixie |
| **Linux Mint** | 21.3, 22, 22.3 | Virginia, Wilma, Zena |
| **Linux Mate** | 24.04 (на базе Ubuntu) | Noble |

## Возможности

- ✅ Автоматическое определение версии дистрибутива (Ubuntu | Noble | 24.04)
- ✅ Управление репозиториями с опцией удаления всех текущих
- ✅ Поддержка формата deb822 (.sources) вместо устаревшего .list
- ✅ Конвертация существующих репозиториев в deb822
- ✅ Поддержка PPA репозиториев с конвертацией в deb822
- ✅ Установка пакетов группами (base/system/desktop/multimedia/network) за один вызов apt
- ✅ Опция обновления всех пакетов (apt_upgrade) с выбором стратегии
- ✅ Теги для выборочного выполнения задач (awesome_repo, awesome_pkg_install)
- ✅ FQCN (Fully Qualified Collection Names) для всех модулей

## Переменные

### Основные настройки

| Переменная | Тип | Значение по умолчанию | Описание |
|------------|-----|----------------------|----------|
| `awesome_remove_all_repos` | bool | `false` | Удалить все существующие репозитории перед добавлением новых |
| `awesome_convert_to_deb822` | bool | `false` | Конвертировать существующие репозитории в формат deb822 |
| `awesome_use_deb822` | bool | `false` | Использовать deb822 для новых репозиториев |
| `apt_upgrade` | string/bool | `false` | Обновить все пакеты (`true`/`false`/`dist`/`full`/`safe`) |
| `apt_autoremove` | bool | `false` | Автоматически удалять ненужные зависимости |
| `apt_cache_valid_time` | int | `3600` | Время кэширования apt в секундах |

### Списки пакетов

Роль разбивает пакеты на логические группы для оптимизированной установки:

| Переменная | Описание | Примеры пакетов |
|------------|----------|----------------|
| `default_awesome_package_list_base` | Базовые системные пакеты | curl, wget, git, vim, htop |
| `default_awesome_package_list_system` | Системные утилиты | build-essential, dkms, ssh |
| `default_awesome_package_list_desktop` | Приложения для рабочего стола | firefox, thunderbird, vlc |
| `default_awesome_package_list_multimedia` | Мультимедиа пакеты | ffmpeg, audacity, gstreamer |
| `default_awesome_package_list_network` | Сетевые утилиты | nmap, wireshark, openvpn |

Для каждого списка есть соответствующая кастомная переменная:
- `custom_awesome_package_list_base`
- `custom_awesome_package_list_system`
- `custom_awesome_package_list_desktop`
- `custom_awesome_package_list_multimedia`
- `custom_awesome_package_list_network`

### Репозитории

| Переменная | Тип | Описание |
|------------|-----|----------|
| `awesome_repo_list` | list | Основной список репозиторий (формируется автоматически) |
| `custom_awesome_repo_list` | list | Пользовательские репозитории |

#### Форматы описания репозиториев

**1. Обычная строка (автоматическая конвертация в deb822):**

```yaml
- "deb http://archive.ubuntu.com/ubuntu jammy main restricted"
```

**2. PPA репозиторий:**

```yaml
- "ppa:nginx/stable"
```

**3. Репозиторий с архитектурой и ключом:**

```yaml
- "deb [arch=amd64 signed-by=/usr/share/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu noble stable"
```

**4. Полный формат deb822 (маппинг):**

```yaml
- type: "deb"
  uris: "https://download.docker.com/linux/ubuntu"
  suites: "noble"
  components: ["stable"]
  signed_by: "/usr/share/keyrings/docker.gpg"
  architectures: ["amd64"]
```

## Автоматически определяемые факты

Роль автоматически определяет следующие факты о дистрибутиве:

```yaml
distro_name: "Ubuntu"                    # Имя дистрибутива
distro_version: "24.04"                  # Версия
distro_major_version: "24"               # Мажорная версия
distro_release: "noble"                  # Кодовое имя релиза
distro_codename: "noble"                 # Псевдоним (для обратной совместимости)
distro_display: "Ubuntu 24.04 (Noble)"   # Человекочитаемое представление
distro_family: "Debian"                  # Семейство ОС
distro_base: "Ubuntu"                    # Базовая ОС (для производных)
```

Для Linux Mint дополнительно определяются:

```yaml
mint_codename: "wilma"                   # Кодовое имя Mint
mint_version: "22"                       # Версия Mint
ubuntu_base_version: "24.04"             # Базовая версия Ubuntu
ubuntu_base_codename: "noble"            # Кодовое имя базовой Ubuntu
```

## Примеры плейбуков

### Базовое использование

```yaml
---
- name: Setup workstation with awesome software
  hosts: all
  become: yes
  gather_facts: yes
  
  roles:
    - nightsnake.ansible-awesome-software
```

### Расширенная настройка с кастомными пакетами и репозиториями

```yaml
---
- name: Advanced workstation setup
  hosts: workstations
  become: yes
  gather_facts: yes
  
  vars:
    # Управление репозиториями
    awesome_remove_all_repos: false
    awesome_convert_to_deb822: true
    awesome_use_deb822: true
    
    # Обновление системы
    apt_upgrade: full
    apt_autoremove: true
    
    # Кастомные пакеты
    custom_awesome_package_list_base:
      - tmux
      - zsh
      - fzf
      - ripgrep
      - fd-find
    
    custom_awesome_package_list_desktop:
      - chromium-browser
      - keepassxc
      - obs-studio
      - slack-desktop
      - discord
    
    custom_awesome_package_list_system:
      - docker.io
      - docker-compose
      - kubectl
      - helm
    
    custom_awesome_package_list_multimedia:
      - spotify-client
      - steam-installer
    
    custom_awesome_package_list_network:
      - remmina
      - wireguard
    
    # Кастомные репозитории
    custom_awesome_repo_list:
      - "ppa:fish-shell/release-3"
      - "deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable"
      - "deb https://packages.cloud.google.com/apt kubernetes-xenial main"
  
  roles:
    - nightsnake.ansible-awesome-software
```

### Только управление репозиториями (без установки пакетов)

```yaml
---
- name: Configure repositories only
  hosts: all
  become: yes
  gather_facts: yes
  
  vars:
    awesome_remove_all_repos: true
    awesome_use_deb822: true
    custom_awesome_repo_list:
      - "deb http://archive.ubuntu.com/ubuntu noble main restricted universe multiverse"
      - "deb http://archive.ubuntu.com/ubuntu noble-updates main restricted universe multiverse"
  
  roles:
    - nightsnake.ansible-awesome-software
  tags: awesome_repo
```

### Только установка пакетов (сохраняя существующие репозитории)

```yaml
---
- name: Install packages only
  hosts: all
  become: yes
  gather_facts: yes
  
  vars:
    custom_awesome_package_list_desktop:
      - firefox
      - thunderbird
      - vlc
  
  roles:
    - nightsnake.ansible-awesome-software
  tags: awesome_pkg_install
```

## Использование тегов

| Тег | Описание |
|-----|----------|
| `awesome_repo` | Выполнить только задачи управления репозиториями |
| `awesome_pkg_install` | Выполнить только установку пакетов |
| `repos` | Алиас для `awesome_repo` |
| `packages` | Алиас для `awesome_pkg_install` |

Пример:

```bash
# Только установка пакетов
ansible-playbook playbook.yml --tags awesome_pkg_install

# Только управление репозиториями
ansible-playbook playbook.yml --tags repos
```

## Требования

- Ansible >= 2.25 (для полной поддержки deb822)
- Python >= 3.8 на целевой системе
- python3-apt (устанавливается автоматически)

## Лицензия

MIT

## Автор

nightsnake (адаптировано и расширено)

## Поддержка

Для багов и предложений создавайте issue на [GitHub](https://github.com/nightsnake/ansible-awesome-software/issues)