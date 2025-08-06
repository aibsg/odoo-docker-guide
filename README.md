# odoo-docker-guide
## Установка Docker

### Для Ubuntu/Linux
```bash
# Установка зависимостей
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Установка Docker
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Проверка установки
sudo docker run hello-world
```
### Для Windows

1. Скачайте [Docker Desktop](https://docs.docker.com/desktop/setup/install/windows-install/) и установите Docker Desktop. Во время установки не меняйте базовые настройки.
2. Установите WSL используя команду PowerShell:
``` PowerShell
wsl --install
```
3. Перезагрузите компьютер 

## Подключение DockerHub 
1. Открываем файл deamon.json. Расположение:
	- Linux: /etc/docker/daemon.json
	- Windows (либо этот): C:\ProgramData\docker\config\daemon.json
	- Windows (либо этот): C:\Users\ИмяПользователя\.docker\daemon.json
	- MacOS: файл редактируется в приложении Docker Desktop
2. В файле будет уже существующая конфигурация. Нужно добавить к ней настройки зеркал (на 03.08.2025 настройки работают).
 ``` json
  "registry-mirrors": [ 
    "https://mirror.gcr.io/", 
    "https://dockerhub.timeweb.cloud"
  ]
```
По итогу конфигурация будет выгядеть примерно так: 
``` json
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "registry-mirrors": [
    "https://mirror.gcr.io/",
    "https://dockerhub.timeweb.cloud"
  ]
}
```
3. Перезапускаем Docker (а лучше всю систему).

## Запуск Odoo

### 1. Создаем структуру проекта
### Для Linux/macOS
``` bash
mkdir odoo-docker && cd odoo-docker
mkdir -p custom-addons config postgres/data
```

### Для Windows (PowerShell)
``` PowerShell
New-Item -ItemType Directory -Path "odoo-docker" -Force
cd odoo-docker
New-Item -ItemType Directory -Path "custom-addons", "config", "postgres\data" -Force
```

### 2. Создаем docker-compose.yaml

``` yml
version: '3'                               # Версия синтаксиса docker-compose

services:
  odoo:                                    # Сервис Odoo
    image: odoo:17.0                       # Используем официальный образ Odoo 17.0
    env_file: .env                         # Загружаем переменные среды из .env файла
    depends_on:
      - postgres                           # Odoo будет ждать запуска PostgreSQL перед стартом
    ports:
      - "127.0.0.1:8069:8069"              # Проброс порта: доступ к Odoo по http://localhost:8069
    volumes:
      - ./custom-addons:/mnt/custom-addons  # Монтируем папку с кастомными модулями
      - ./config/odoo.conf:/etc/odoo/odoo.conf  # Подключаем собственный конфигурационный файл

  postgres:                                # Сервис базы данных PostgreSQL
    image: postgres:13                     # Используем официальный образ PostgreSQL 13
    env_file: .env                         # Загружаем переменные среды из того же .env
    volumes:
      - ./postgres/data:/var/lib/postgresql/data  # Монтируем локальный том для хранения данных БД
```

### 3. Создаем .env
Файл должен находится в корневой директории
``` conf
POSTGRES_DB=postgres               # Название БД (по умолчанию — postgres)
POSTGRES_USER=odoo                 # Имя пользователя для подключения к БД
POSTGRES_PASSWORD=your_secure_password   # Пароль для пользователя
PGDATA=/var/lib/postgresql/data   # Путь, где PostgreSQL будет хранить данные внутри контейнера
```
### 4. Создаем config/odoo.conf
Файл должен находится в папке config
``` conf
[options]
addons_path = /mnt/custom-addons      # Путь до папки с пользовательскими модулями внутри контейнера
db_host = postgres                    # Имя сервиса БД (используется Docker-сеть)
db_user = odoo                        # Пользователь для подключения к БД (должен совпадать с .env)
db_password = your_secure_password    # Пароль пользователя (также из .env)
```
### 5. Запускаем контейнеры

``` bash
docker-compose up -d
```

После запуска Odoo будет доступен по адресу:  
[http://localhost:8069](http://localhost:8069)

## Добавление пользовательских модулей

Структура описанная выше подразумевает под собой добавление новых модулей. 
Чтобы добавить новые модули нужно переметить их в папку */custom-addons*. Далее нужно будет в самом приложении обновить список модулей и появится возможность их загрузить.
