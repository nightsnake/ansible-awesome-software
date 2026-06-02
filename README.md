# Ansible Role: Awesome Software Pack

Ansible роль для установки репозиториев и программного обеспечения на Debian/Ubuntu/Linux Mint системы. Роль оптимизирована для автоматизации настройки рабочей станции с поддержкой формата deb822, установкой DEB-пакетов по URL и интеллектуальным определением типа системы.

## Поддерживаемые дистрибутивы

Дистрибутив | Версии | Кодовые имена  
**---|---**  
**Ubuntu** | 22.04, 24.04, 26.04 | Jammy, Noble, Wily  
**Debian** | 12, 13 | Bookworm, Trixie  
**Linux Mint** | 21.3, 22, 22.3 | Virginia, Wilma, Zena  

## Возможности

  * ✅ Автоматическое определение версии дистрибутива (Ubuntu | Noble | 24.04)
  * ✅ Автоматическое определение типа системы (виртуальная машина/контейнер/bare metal)
  * ✅ Автоматическая установка qemu-guest-agent на виртуальные машины
  * ✅ Управление репозиториями с опцией удаления всех текущих
  * ✅ Поддержка формата deb822 (.sources) наряду с legacy форматом (.list)
  * ✅ Установка репозиториев через DEB-пакет по прямой ссылке
  * ✅ Конвертация существующих репозиториев в deb822
  * ✅ Установка пакетов группами (base/system/desktop/multimedia/network) за один вызов apt
  * ✅ Опция обновления всех пакетов (apt_upgrade) с выбором стратегии
  * ✅ Теги для выборочного выполнения задач (awesome_repo, awesome_pkg_install)
  * ✅ Режим отладки для диагностики проблем

## Переменные

### Основные настройки

Переменная | Тип | Значение по умолчанию | Описание  
**---|---**  
`awesome_debug` | bool | `false` | Включить подробный вывод отладки  
`awesome_remove_all_repos` | bool | `false` | Удалить все существующие репозитории перед добавлением новых  
`awesome_convert_to_deb822` | bool | `false` | Конвертировать существующие репозитории в формат deb822  
`awesome_install_qemu_agent` | bool | `true` | Установить qemu-guest-agent на виртуальные машины  
`apt_upgrade` | string/bool | `false` | Обновить все пакеты (`true`/`false`/`dist`/`full`/`safe`)  
`apt_autoremove` | bool | `false` | Автоматически удалять ненужные зависимости  
`apt_cache_valid_time` | int | `3600` | Время кэширования apt в секундах  

### Списки пакетов

Роль разбивает пакеты на логические группы для оптимизированной установки:

Переменная | Описание | Примеры пакетов  
**---|---**  
`default_awesome_package_list_base` | Базовые системные пакеты | curl, wget, git, vim, htop  
`default_awesome_package_list_system` | Системные утилиты | build-essential, dkms, ssh  
`default_awesome_package_list_desktop` | Приложения для рабочего стола | firefox, thunderbird, vlc  
`default_awesome_package_list_multimedia` | Мультимедиа пакеты | ffmpeg, audacity, gstreamer  
`default_awesome_package_list_network` | Сетевые утилиты | nmap, wireshark, openvpn  

Для каждого списка есть соответствующая кастомная переменная:

  * `custom_awesome_package_list_base`
  * `custom_awesome_package_list_system`
  * `custom_awesome_package_list_desktop`
  * `custom_awesome_package_list_multimedia`
  * `custom_awesome_package_list_network`

### Репозитории по умолчанию

Роль автоматически добавляет стандартные репозитории для вашего дистрибутива:

  * **Debian** : main, security, updates
  * **Ubuntu** : main, restricted, universe, multiverse, updates, backports, security
  * **Linux Mint** : репозитории Mint + базовые репозитории Ubuntu

Дефолтные репозитории записываются в файл `/etc/apt/sources.list.d/дистрибутив.list` (например, `debian.list` для Debian).

### Пользовательские репозитории

Переменная | Тип | Описание  
**---|---**  
`custom_awesome_repo_list` | list | Пользовательские репозитории  

Пользовательские репозитории задаются через словарь в одном из **трёх** форматов:

#### Формат 1: LEGACY (создаётся файл `.list`)

Поля:
  * `repo` (обязательное): строка с описанием репозитория (deb-строка или ppa:имя)
  * `file` (обязательное): имя файла без расширения (будет создан `file.list`)

```yaml
- repo: "deb http://deb.debian.org/debian trixie main contrib non-free"
  file: "debian_main"

- repo: "ppa:nginx/stable"
  file: "nginx"

- repo: "deb [arch=amd64] https://download.docker.com/linux/debian trixie stable"
  file: "docker"
```

#### Формат 2: DEB822 (создаётся файл `.sources`)

Поля:
  * `name` (обязательное): имя файла без расширения (будет создан `name.sources`)
  * `type` (опциональное): тип репозитория, обычно 'deb'
  * `uris` (обязательное): URI репозитория
  * `suites` (обязательное): кодовое имя релиза (например, 'trixie', 'noble')
  * `components` (обязательное): список компонентов (например, ['main', 'universe'])
  * `signed_by` (опциональное): путь к ключу GPG
  * `architectures` (опциональное): список архитектур

```yaml
- name: "zabbix"
  type: "deb"
  uris: "https://repo.zabbix.com/zabbix/7.0/debian/"
  suites: "trixie"
  components: ["main"]
  architectures: ["amd64"]
  signed_by: "/etc/apt/keyrings/zabbix-repo.asc"

- name: "grafana"
  type: "deb"
  uris: "https://packages.grafana.com/oss/deb"
  suites: "stable"
  components: ["main"]
  signed_by: "/etc/apt/keyrings/grafana.gpg"
```

#### Формат 3: DEB-ПАКЕТ ПО ССЫЛКЕ (установка репозитория через .deb)

Поля:
  * `url` (обязательное): прямая ссылка на DEB-пакет с репозиторием

```yaml
- url: "https://example.com/pool/main/third-party-repo.deb"
- url: "https://packages.microsoft.com/config/debian/12/packages-microsoft-prod.deb"
```

**Важно:** APT сам скачает и установит пакет по указанной ссылке. Этот метод полезен для репозиториев, которые распространяются в виде DEB-пакета (Microsoft, Google, Docker и др.).

## Автоматическое определение типа системы

Роль автоматически определяет, где запущена система, используя факты Ansible:

Тип системы | Определение | Действие роли
**|---**  
**Виртуальная машина** | KVM, Xen, VMware, VirtualBox и др. | Устанавливает и запускает `qemu-guest-agent`
**Контейнер** | Docker, LXC, LXD, Podman | Пропускает установку агента
**Bare metal** | Физический сервер/ПК | Пропускает установку агента

Для виртуальных машин роль не только устанавливает, но и включает сервис `qemu-guest-agent`, чтобы он стартовал автоматически.

## Автоматически определяемые факты

Роль автоматически определяет следующие факты о дистрибутиве:

```yaml
distro_name: "Debian"               # Имя дистрибутива
distro_version: "13.4"              # Полная версия
distro_major_version: "13"          # Мажорная версия
distro_release: "trixie"            # Кодовое имя релиза
distro_codename: "trixie"           # Псевдоним (для обратной совместимости)
distro_display: "Debian 13 (Trixie)" # Человекочитаемое представление
distro_family: "Debian"             # Семейство ОС
distro_base: "Debian"               # Базовая ОС (для производных)
distro_normalized_version: "13"     # Нормализованная версия (без минорной)
```

Дополнительные факты о виртуализации:

```yaml
system_virt_type: "kvm"             # Тип виртуализации
system_virt_role: "guest"           # Роль (guest/host)
is_virtual_machine: true            # Флаг виртуальной машины
is_container: false                 # Флаг контейнера
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

    # Обновление системы
    apt_upgrade: full
    apt_autoremove: true

    # Кастомные пакеты
    custom_awesome_package_list_base:
      - tmux
      - zsh
      - fzf
      - ripgrep

    custom_awesome_package_list_desktop:
      - chromium-browser
      - keepassxc
      - obs-studio

    custom_awesome_package_list_system:
      - docker.io
      - kubectl
      - helm

    # Кастомные репозитории (все три формата)
    custom_awesome_repo_list:
      # Legacy формат
      - repo: "ppa:fish-shell/release-3"
        file: "fish"

      # Deb822 формат
      - name: "zabbix"
        type: "deb"
        uris: "https://repo.zabbix.com/zabbix/7.0/debian/"
        suites: "{{ ansible_distribution_release }}"
        components: ["main"]
        signed_by: "/etc/apt/keyrings/zabbix-repo.asc"

      # DEB-пакет по ссылке
      - url: "https://packages.microsoft.com/config/debian/12/packages-microsoft-prod.deb"

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
    custom_awesome_repo_list:
      - repo: "deb http://archive.ubuntu.com/ubuntu noble main restricted universe multiverse"
        file: "ubuntu_main"
      - repo: "deb http://archive.ubuntu.com/ubuntu noble-updates main restricted universe multiverse"
        file: "ubuntu_updates"
      - url: "https://example.com/custom-repo.deb"

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

### С включённым режимом отладки

```yaml
---
- name: Debug repository configuration
  hosts: backup07
  become: yes
  gather_facts: yes

  vars:
    awesome_debug: true
    custom_awesome_repo_list:
      - repo: "deb http://deb.debian.org/debian trixie main contrib non-free"
        file: "debian_main"

  roles:
    - nightsnake.ansible-awesome-software
```

## Использование тегов

Тег | Описание  
**|---**  
`awesome_repo` | Выполнить только задачи управления репозиториями  
`awesome_pkg_install` | Выполнить только установку пакетов  
`repos` | Алиас для `awesome_repo`  
`packages` | Алиас для `awesome_pkg_install`  

Пример:

```bash
# Только установка пакетов
ansible-playbook playbook.yml --tags awesome_pkg_install

# Только управление репозиториями
ansible-playbook playbook.yml --tags repos
```

## Требования

  * Ansible >= 2.25 (для полной поддержки deb822)
  * Python >= 3.8 на целевой системе
  * python3-apt (устанавливается автоматически)

## Лицензия

MIT

## Автор

nightsnake (адаптировано и расширено)

## Поддержка

Для багов и предложений создавайте issue на GitHub