# Установка и настройка Debian на WSL с поддержкой NVIDIA CUDA

**📝 Итоговая инструкция по установке и настройке Debian на WSL (Версия 7 - Стиль V2 + Репозиторий NVIDIA CUDA, 17 Апреля 2025)**  
*Для Windows 11, Intel Core i7 13700K, NVIDIA RTX 4090*

## Содержание

- [1. Подготовка Windows 11](#1-подготовка-windows-11)
- [2. Установка Debian WSL](#2-установка-debian-wsl)
- [3. Базовая настройка Debian](#3-базовая-настройка-debian)
- [4. Создание пользователя](#4-создание-пользователя)
- [5. Установка компонентов NVIDIA](#5-установка-компонентов-nvidia)
- [6. Установка ПО](#6-установка-по)
- [7. Проверка системы](#7-проверка-системы)
- [8. Финализация](#8-финализация)
- [9. Устранение типичных проблем](#9-устранение-типичных-проблем)
- [10. Удаление WSL-дистрибутива](#10-удаление-wsl-дистрибутива)
- [Важные примечания](#-важные-примечания)
- [Актуальные ссылки](#-актуальные-ссылки)

---

## 1. Подготовка Windows 11

*(Выполнять в Windows PowerShell от имени администратора)*

### 1.1 Включение компонентов

1.  **Виртуализация**:
    * В BIOS/UEFI активируйте технологию виртуализации (например, **Intel Virtualization Technology (VT-x)** или **AMD-V**).
    * В Windows:
        * `Win + R` → `optionalfeatures` → убедитесь, что включены:
            * **Платформа виртуальной машины (Virtual Machine Platform)**
            * **Подсистема Windows для Linux (Windows Subsystem for Linux)**
        * *(Опционально)* Компонент **Hyper-V** может быть включен, он также содержит "Платформу виртуальной машины", но не является строго обязательным *только* для WSL2.

2.  **Обновление WSL**:
```powershell
wsl --update
wsl --set-default-version 2
```

## 2. Установка Debian WSL

*(Выполнять в Windows PowerShell)*

### 2.1 Вариант 1: Установка из Microsoft Store

1.  Откройте Microsoft Store и найдите "Debian"
2.  Нажмите "Получить" и дождитесь завершения установки
3.  После установки нажмите "Запустить" и дождитесь завершения первоначальной настройки
4.  Создайте пользователя и пароль при первом запуске

### 2.2 Вариант 2: Ручная установка с выбором расположения

```powershell
# Шаг 1: Установка Debian из Microsoft Store без запуска
wsl --install debian --no-launch

# Шаг 2: Создание папки для хранения дистрибутива (пример)
mkdir -p C:\WSL\Debian

# Шаг 3: Экспорт установленного образа в tar-файл
wsl --export Debian C:\WSL\debian-backup.tar

# Шаг 4: Удаление исходного дистрибутива
wsl --unregister Debian

# Шаг 5: Импорт в выбранное местоположение
wsl --import Debian C:\WSL\Debian C:\WSL\debian-backup.tar --version 2

# Шаг 6: Удаление временного tar-файла
Remove-Item C:\WSL\debian-backup.tar

# Шаг 7: Запустите Debian и создайте пользователя
wsl -d Debian
# При первом запуске может потребоваться настроить пользователя по умолчанию вручную
# (см. раздел 4, если пользователь не был создан автоматически)
```

### 2.3 Изменение местоположения уже установленного дистрибутива

```powershell
# Остановите дистрибутив
wsl --terminate Debian

# Экспортируйте его в tar-файл
wsl --export Debian C:\temp\debian-backup.tar

# Удалите исходный дистрибутив
wsl --unregister Debian

# Импортируйте его в новое местоположение
wsl --import Debian C:\WSL\Debian C:\temp\debian-backup.tar --version 2

# Можно удалить временный tar-файл
Remove-Item C:\temp\debian-backup.tar
```

### 2.4 Запуск Debian WSL

```powershell
# Запуск Debian
wsl -d Debian
```

## 3. Базовая настройка Debian

*(Выполнять в терминале Debian WSL)*

### 3.1 Обновление репозиториев и системы

```bash
# Установка обновлений
sudo apt update
sudo apt upgrade -y
```

```bash
# Установка базовых пакетов для работы с репозиториями и сетью
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release wget
```

```bash
# Включение contrib, non-free и non-free-firmware репозиториев (важно для драйверов и ПО)
sudo sed -i 's/main/main contrib non-free non-free-firmware/g' /etc/apt/sources.list
```

```bash
# Обновление после добавления репозиториев
sudo apt update
```

### 3.2 Локализация и время

```bash
# Установка необходимых пакетов
sudo apt install -y locales tzdata
```

```bash
# Настройка локализации (раскомментируем русскую и английскую UTF-8)
sudo sed -i 's/# ru_RU.UTF-8 UTF-8/ru_RU.UTF-8 UTF-8/' /etc/locale.gen
sudo sed -i 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen
sudo locale-gen
```

```bash
# Настройка системного языка по умолчанию
sudo update-locale LANG=ru_RU.UTF-8
```

```bash
# Настройка глобальных переменных окружения (основной метод)
echo "LANG=ru_RU.UTF-8" | sudo tee /etc/environment
echo "LC_ALL=ru_RU.UTF-8" | sudo tee -a /etc/environment
echo "LANGUAGE=ru_RU:ru" | sudo tee -a /etc/environment
```

```bash
# Установка часового пояса (пример: Москва) - Используем современный метод
sudo timedatectl set-timezone Europe/Moscow
```

### 3.3 Настройка WSL (`/etc/wsl.conf` и `.wslconfig`)

1.  **Создание файла `/etc/wsl.conf` внутри Debian:**
    * Этот файл управляет поведением конкретного дистрибутива WSL.
    ```bash
    # Создание файла настроек WSL с использованием heredoc (стиль V2)
    sudo tee /etc/wsl.conf > /dev/null << EOL
[user]
default=wsluser # Убедитесь, что пользователь с таким именем будет создан (Раздел 4)

[interop]
enabled=true
appendWindowsPath=true

[boot]
systemd=true # Важно для Docker и других служб

[network]
generateResolvConf = true
EOL
    ```
    * **Примечание**: После изменения `/etc/wsl.conf` нужен перезапуск WSL (`wsl --shutdown` в PowerShell).

2.  **Создание/Изменение файла `.wslconfig` на Windows:**
    * Этот файл (`%USERPROFILE%\.wslconfig`) управляет глобальными настройками WSL2 для *всех* дистрибутивов.
    * Создайте или отредактируйте файл `C:\Users\<Ваше_Имя_Пользователя>\.wslconfig` (замените `<Ваше_Имя_Пользователя>`) со следующим содержимым:
    ```ini
    [wsl2]
    # Настройки ресурсов (примеры, измените под свои нужды)
    memory=16GB          # Например, 16 ГБ ОЗУ
    processors=8         # Например, 8 процессорных ядер
    # swap=4GB             # Можно указать файл подкачки, если нужно

    # Настройки для поддержки GUI приложений (WSLg)
    guiApplications=true

    # Параметры ядра для совместимости (могут быть нужны для systemd/cgroups)
    kernelCommandLine = cgroup_no_v1=all systemd.unified_cgroup_hierarchy=1

    # Отладка (если нужно)
    # debugShell=true
    ```
    * **Примечание**: После создания/изменения `.wslconfig` нужен перезапуск WSL (`wsl --shutdown` в PowerShell).

## 4. Создание пользователя

*(Выполнять в терминале Debian WSL)*

```bash
# Установка fish shell и sudo
sudo apt install -y fish sudo
```

```bash
# Установка fish shell по умолчанию для root
sudo chsh -s /usr/bin/fish root
```

```bash
# Создание пользователя wsluser (если не создан при первом запуске)
# Если пользователь уже есть, эта команда выдаст ошибку, но это не страшно.
sudo useradd -m -G sudo -s /usr/bin/fish wsluser || echo "Пользователь wsluser, возможно, уже существует."
```

```bash
# Установка/смена пароля для пользователя wsluser
echo "Установите или смените пароль для wsluser:"
sudo passwd wsluser
```

```bash
# Настройка sudo без пароля (удобно, но менее безопасно)
echo "wsluser ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/wsluser
sudo chmod 440 /etc/sudoers.d/wsluser
```
*Примечание: Убедитесь, что `default=wsluser` установлен в `/etc/wsl.conf`.*

## 5. Установка компонентов NVIDIA

*(Выполнять в терминале Debian WSL)*

**Важное примечание**:
* Убедитесь, что на **Windows** установлен **последний драйвер NVIDIA** для вашей видеокарты (RTX 4090) с [официального сайта](https://www.nvidia.com/Download/index.aspx). Сам драйвер в Debian **не устанавливается**.
* Мы будем использовать официальный репозиторий NVIDIA для WSL (`.deb`), а не пакет `nvidia-cuda-toolkit` из репозитория Debian, так как он обычно содержит более свежую версию CUDA и корректно устанавливается в WSL.

### 5.1 Установка NVIDIA Container Toolkit (для Docker GPU)

```bash
# Добавление официального GPG-ключа NVIDIA Container Toolkit
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
```

```bash
# Добавление репозитория NVIDIA Container Toolkit
curl -fsSL https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sudo sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

```bash
# Обновление списка пакетов
sudo apt update
```

```bash
# Установка NVIDIA Container Toolkit
sudo apt install -y nvidia-container-toolkit
```

### 5.2 Установка CUDA Toolkit (Инструменты разработчика) через репозиторий NVIDIA

* *Примечание:* Команды ниже соответствуют установке CUDA Toolkit 12.4 (на момент Апреля 2025). **Всегда проверяйте актуальные команды** на [официальной странице загрузки CUDA для WSL-Ubuntu](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=WSL-Ubuntu&target_version=2.0&target_type=deb_network), так как версии и URL могут меняться. Выбирайте **deb (network)**.

```bash
# Скачиваем пиннинг-файл для репозитория CUDA (указывает приоритет)
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
```

```bash
# Перемещаем пиннинг-файл
sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600
```

```bash
# Скачиваем deb-пакет для добавления репозитория CUDA (пример для CUDA 12.4, проверьте актуальную версию!)
# Используем сетевой установщик репозитория
CUDA_VERSION_MAJOR_MINOR="12.4"
CUDA_REPO_DEB="cuda-repo-wsl-ubuntu-${CUDA_VERSION_MAJOR_MINOR//./-}_1.0-1_amd64.deb" # Имя может чуть отличаться
wget "https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/${CUDA_REPO_DEB}"
```

```bash
# Устанавливаем скачанный пакет репозитория
sudo dpkg -i ${CUDA_REPO_DEB}
```

```bash
# Копируем ключ из установленного репозитория (путь может зависеть от версии в имени пакета)
# Предполагается, что ключ лежит в /var/cuda-repo-wsl-ubuntu-12-4/
sudo cp "/var/cuda-repo-wsl-ubuntu-${CUDA_VERSION_MAJOR_MINOR//./-}/cuda-"*"-keyring.gpg" /usr/share/keyrings/
```

```bash
# Обновляем список пакетов после добавления репозитория CUDA
sudo apt update
```

```bash
# Устанавливаем CUDA Toolkit (мета-пакет)
# Это НЕ УСТАНАВЛИВАЕТ драйвер, только сам Toolkit.
sudo apt install -y cuda-toolkit
```

```bash
# Очистка (удаление скачанного deb-файла репозитория)
rm ${CUDA_REPO_DEB}
```

### 5.3 Настройка переменных окружения CUDA для Fish Shell

*(Этот шаг необходим для работы `nvcc` и других утилит из командной строки)*

```bash
# --- Для пользователя wsluser ---
# Добавляем пути в config.fish пользователя (выполнять от имени wsluser, НЕ под sudo)
# Используем `echo ... >> file` для добавления строк
echo '' >> ~/.config/fish/config.fish
echo '# Add CUDA paths to environment' >> ~/.config/fish/config.fish
echo 'if test -d /usr/local/cuda' >> ~/.config/fish/config.fish
echo '    set -xp PATH /usr/local/cuda/bin $PATH' >> ~/.config/fish/config.fish
echo '    set -xp LD_LIBRARY_PATH /usr/local/cuda/lib64 $LD_LIBRARY_PATH' >> ~/.config/fish/config.fish
echo 'end' >> ~/.config/fish/config.fish
```

```bash
# --- Для пользователя root ---
# Используем `sudo bash -c 'echo "..." >> file'` для добавления строк от имени root
sudo bash -c 'echo "" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "# Add CUDA paths to environment" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "if test -d /usr/local/cuda" >> /root/.config/fish/config.fish'
# Одинарные кавычки вокруг всей строки echo для bash -c, чтобы $PATH раскрылся позже в fish
sudo bash -c 'echo '\''    set -xp PATH /usr/local/cuda/bin $PATH'\'' >> /root/.config/fish/config.fish'
sudo bash -c 'echo '\''    set -xp LD_LIBRARY_PATH /usr/local/cuda/lib64 $LD_LIBRARY_PATH'\'' >> /root/.config/fish/config.fish'
sudo bash -c 'echo "end" >> /root/.config/fish/config.fish'
```
*Примечание: Перезапустите WSL или сессию Fish, чтобы переменные применились.*

## 6. Установка ПО

*(Выполнять в терминале Debian WSL от имени пользователя wsluser)*

### 6.1 Системные утилиты

```bash
# Установка базовых утилит
sudo apt install -y \
    nano \
    python3 \
    python3-pip \
    python3-venv \
    htop \
    curl \
    wget \
    unzip \
    git \
    build-essential \
    pkg-config \
    fzf \
    fd-find \
    bat # Устанавливаем сразу, чтобы алиасы работали
```

### 6.2 Fish Shell с популярными плагинами

```bash
# Создание директорий для конфигурации пользователя
mkdir -p ~/.config/fish/functions
mkdir -p ~/.config/fish/completions
```

```bash
# --- Настройка fish для пользователя wsluser ---
echo '# --- Fish Shell Config WSL Debian ---' > ~/.config/fish/config.fish
echo '' >> ~/.config/fish/config.fish
echo '# Алиасы' >> ~/.config/fish/config.fish
echo "alias ll='ls -la'" >> ~/.config/fish/config.fish
echo "alias la='ls -A'" >> ~/.config/fish/config.fish
echo "alias l='ls'" >> ~/.config/fish/config.fish
echo "alias cls='clear'" >> ~/.config/fish/config.fish
echo "alias ..='cd ..'" >> ~/.config/fish/config.fish
echo "alias ...='cd ../..'" >> ~/.config/fish/config.fish
echo '' >> ~/.config/fish/config.fish
echo '# Улучшенные утилиты (требуют установки bat, fd-find)' >> ~/.config/fish/config.fish
echo "type -q batcat && alias cat='batcat --paging=never'" >> ~/.config/fish/config.fish # Для Debian bat называется batcat
echo "type -q fdfind && alias fd='fdfind'" >> ~/.config/fish/config.fish # fd называется fdfind
echo "type -q fdfind && alias find='fdfind'" >> ~/.config/fish/config.fish # Можно и find заменить
echo '' >> ~/.config/fish/config.fish
echo '# Настройка fish' >> ~/.config/fish/config.fish
echo 'set -U fish_greeting ""' >> ~/.config/fish/config.fish # Отключаем стандартное приветствие
echo 'set fish_key_bindings fish_default_key_bindings' >> ~/.config/fish/config.fish
echo 'set fish_autosuggestion_enabled 1' >> ~/.config/fish/config.fish
echo '' >> ~/.config/fish/config.fish
echo '# FZF интеграция (расширенная)' >> ~/.config/fish/config.fish
echo 'set -gx FZF_DEFAULT_COMMAND "fdfind --type f --color=always --strip-cwd-prefix 2>/dev/null || find . -type f"' >> ~/.config/fish/config.fish
echo 'set -gx FZF_CTRL_T_COMMAND $FZF_DEFAULT_COMMAND' >> ~/.config/fish/config.fish
echo 'set -gx FZF_CTRL_T_OPTS "--preview '\''batcat --color=always --style=numbers --line-range :500 {}'\''"' >> ~/.config/fish/config.fish
echo 'set -gx FZF_ALT_C_COMMAND "fdfind --type d --color=always --strip-cwd-prefix 2>/dev/null || find . -type d"' >> ~/.config/fish/config.fish
echo 'set -gx FZF_ALT_C_OPTS "--preview '\''tree -C {} | head -n 50'\''"' >> ~/.config/fish/config.fish
```

```bash
# Создание приветственного сообщения для пользователя
echo 'function fish_greeting' > ~/.config/fish/functions/fish_greeting.fish
echo '    set -l date_str (date "+%Y-%m-%d %H:%M")' >> ~/.config/fish/functions/fish_greeting.fish
echo '    echo "🐧 WSL Debian User - $date_str"' >> ~/.config/fish/functions/fish_greeting.fish
echo 'end' >> ~/.config/fish/functions/fish_greeting.fish
```

```bash
# Настройка завершений для Docker
curl -sL https://raw.githubusercontent.com/docker/cli/master/contrib/completion/fish/docker.fish -o ~/.config/fish/completions/docker.fish
curl -sL https://raw.githubusercontent.com/docker/compose/master/contrib/completion/fish/docker-compose.fish -o ~/.config/fish/completions/docker-compose.fish
```

```bash
# Установка Fisher и плагинов (выполнять от имени пользователя wsluser)
fish -c "curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher"
fish -c "fisher install jethrokuan/z"
fish -c "fisher install PatrickF1/fzf.fish" # Интеграция fzf
fish -c "fisher install jorgebucaran/autopair.fish"
fish -c "fisher install franciscolourenco/done"
fish -c "fisher install edc/bass"
```

```bash
# Установка Starship (выполнять от имени пользователя wsluser)
curl -sS https://starship.rs/install.sh | sh -s -- -y
echo 'starship init fish | source' >> ~/.config/fish/config.fish
```

```bash
# --- Настройка fish для root ---
sudo mkdir -p /root/.config/fish/functions
sudo mkdir -p /root/.config/fish/completions
```

```bash
# Используем `sudo bash -c 'echo "..." >> file'` для добавления строк
sudo bash -c 'echo "# --- Fish Shell Config WSL Debian [ROOT] ---" > /root/.config/fish/config.fish'
sudo bash -c 'echo "" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "# Алиасы" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "alias ll='\''ls -la'\''" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "alias la='\''ls -A'\''" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "alias l='\''ls'\''" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "alias cls='\''clear'\''" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "alias ..='\''cd ..'\''" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "alias ...='\''cd ../..'\''" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "# Улучшенные утилиты" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "type -q batcat && alias cat='\''batcat --paging=never'\''" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "type -q fdfind && alias fd='\''fdfind'\''" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "type -q fdfind && alias find='\''fdfind'\''" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "# Настройка fish" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "set -U fish_greeting \"\"" >> /root/.config/fish/config.fish' # Отключаем стандартное приветствие
sudo bash -c 'echo "set fish_key_bindings fish_default_key_bindings" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "set fish_autosuggestion_enabled 1" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "# FZF интеграция (расширенная)" >> /root/.config/fish/config.fish'
sudo bash -c 'echo '\''set -gx FZF_DEFAULT_COMMAND "fdfind --type f --color=always --strip-cwd-prefix 2>/dev/null || find . -type f"'\'' >> /root/.config/fish/config.fish'
sudo bash -c 'echo '\''set -gx FZF_CTRL_T_COMMAND $FZF_DEFAULT_COMMAND'\'' >> /root/.config/fish/config.fish'
sudo bash -c 'echo '\''set -gx FZF_CTRL_T_OPTS "--preview '\''batcat --color=always --style=numbers --line-range :500 {}'\''"'\'' >> /root/.config/fish/config.fish'
sudo bash -c 'echo '\''set -gx FZF_ALT_C_COMMAND "fdfind --type d --color=always --strip-cwd-prefix 2>/dev/null || find . -type d"'\'' >> /root/.config/fish/config.fish'
sudo bash -c 'echo '\''set -gx FZF_ALT_C_OPTS "--preview '\''tree -C {} | head -n 50'\''"'\'' >> /root/.config/fish/config.fish'
sudo bash -c 'echo "" >> /root/.config/fish/config.fish'
```

```bash
# Установка Starship для root (если еще не установлен глобально) и добавление в конфиг
sudo sh -c 'curl -sS https://starship.rs/install.sh | sh -s -- -y'
sudo bash -c 'echo "starship init fish | source" >> /root/.config/fish/config.fish'
```

```bash
# Приветствие для root
sudo bash -c 'echo "function fish_greeting" > /root/.config/fish/functions/fish_greeting.fish'
sudo bash -c 'echo '\''    set -l date_str (date "+%Y-%m-%d %H:%M")'\'' >> /root/.config/fish/functions/fish_greeting.fish'
sudo bash -c 'echo '\''    echo "🔥 WSL Debian [ROOT] - $date_str"'\'' >> /root/.config/fish/functions/fish_greeting.fish'
sudo bash -c 'echo "end" >> /root/.config/fish/functions/fish_greeting.fish'
```

```bash
# Копирование автозавершений Docker для root (если Docker используется под root)
sudo cp ~/.config/fish/completions/docker.fish /root/.config/fish/completions/ 2>/dev/null || true
sudo cp ~/.config/fish/completions/docker-compose.fish /root/.config/fish/completions/ 2>/dev/null || true
```

### 6.3 Docker с поддержкой GPU

```bash
# Добавление официального GPG-ключа Docker
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

```bash
# Добавление репозитория Docker
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```bash
# Установка Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

```bash
# Настройка Docker для использования NVIDIA GPU по умолчанию
# Убедитесь, что nvidia-container-toolkit установлен (см. Раздел 5.1)
sudo nvidia-ctk runtime configure --runtime=docker
# Эта команда должна автоматически настроить /etc/docker/daemon.json
# Проверьте содержимое файла: cat /etc/docker/daemon.json
```

```bash
# Добавление пользователя в группу docker (чтобы не использовать sudo для docker)
sudo usermod -aG docker wsluser
```

```bash
# Включение и запуск службы Docker через systemd
sudo systemctl enable docker
sudo systemctl start docker
```

```bash
# Проверка статуса Docker
sudo systemctl status docker
```
**Примечание**: После добавления пользователя в группу `docker`, нужно **выйти из сессии WSL и войти снова** (или перезапустить WSL: `wsl --shutdown`), чтобы изменения вступили в силу и `docker` можно было использовать без `sudo`.

## 7. Проверка системы

*(Выполнять в терминале Debian WSL от имени пользователя wsluser)*
* **Важно:** Перезапустите WSL (`wsl --shutdown` в PowerShell, затем `wsl -d Debian`) или хотя бы выйдите и снова войдите в сессию Debian WSL, чтобы применились все настройки групп и переменных окружения.

### 7.1 NVIDIA и CUDA

```bash
# 1. Проверка доступности NVIDIA GPU (драйвер Windows)
nvidia-smi
```

```bash
# 2. Проверка версии CUDA Toolkit (nvcc) - должна работать после установки из репо NVIDIA и настройки env vars
nvcc --version
```

```bash
# 3. Проверка Docker с NVIDIA GPU
# Убедитесь, что Docker запущен: sudo systemctl status docker
docker run --rm --gpus all nvidia/cuda:latest-base nvidia-smi
```
*Если `nvcc --version` не работает, вернитесь к разделу 5.3 и проверьте настройку переменных окружения и перезапуск сессии. Если `docker run ...` требует `sudo`, выйдите и войдите снова.*

### 7.2 Локализация и время

```bash
# Проверка настроек локализации (ожидается ru_RU.UTF-8)
locale
```

```bash
# Проверка часового пояса (ожидается Europe/Moscow или ваш)
timedatectl | grep "Time zone"
```

### 7.3 Fish и Docker

```bash
# Проверка текущей оболочки (ожидается /usr/bin/fish)
echo $SHELL
```

```bash
# Проверка работы Docker без sudo (после перезапуска WSL или newgrp docker)
docker ps
```
*Проверка автозавершения Fish: Введите `docker ` (с пробелом) и нажмите `Tab`. Введите `docker run --gpus ` (с пробелом) и нажмите `Tab`.*

## 8. Финализация

*(Выполнять в терминале Debian WSL)*

```bash
# Финальное обновление системы
sudo apt update
sudo apt upgrade -y
```

```bash
# Удаление ненужных пакетов
sudo apt autoremove -y
```

```bash
# Очистка кеша apt
sudo apt clean
```

```bash
# Сообщение о завершении
echo "Настройка Debian WSL завершена!"
```
*Рекомендуется еще раз перезапустить WSL (`wsl --shutdown` в PowerShell) для чистоты.*

## 9. Устранение типичных проблем

*(Команды для выполнения в терминале Debian WSL)*

```bash
# Исправление прав доступа к конфигам fish пользователя (если создавали с sudo)
sudo chown -R wsluser:wsluser /home/wsluser/.config/fish
chmod -R u+rwX /home/wsluser/.config/fish
```

```bash
# Обновление индекса завершений fish (если автодополнение не работает)
fish -c "fish_update_completions"
```
* **Проблемы с шрифтами (Starship/Powerline):** Установите шрифт с поддержкой Nerd Font (MesloLGS NF, FiraCode NF, Caskaydia Cove NF и т.д.) в Windows и выберите его в настройках Windows Terminal для профиля Debian.
* **Проблемы с русской локализацией:**
    ```bash
    locale -a | grep ru_RU.UTF-8 # Проверить доступность
    cat /etc/environment # Проверить содержимое файла
    # Перезапустите WSL.
    ```
* **Проблемы с CUDA (`nvcc: command not found`):**
    ```bash
    echo $PATH # Проверить наличие /usr/local/cuda/bin
    echo $LD_LIBRARY_PATH # Проверить наличие /usr/local/cuda/lib64
    ls -l /usr/local/cuda # Проверить ссылку
    ls /usr/local/cuda/bin/nvcc # Проверить наличие файла
    # Перепроверьте раздел 5.3, перезапустите сессию/WSL.
    ```
* **Проблемы с иконкой Debian в Windows Terminal:** Откройте настройки Windows Terminal (Ctrl+,) -> Открыть JSON-файл, найдите профиль Debian и исправьте/удалите неверную строку `"icon"`.

## 10. Удаление WSL-дистрибутива

*(Выполнять в Windows PowerShell от имени администратора)*

```powershell
# Остановка работающего дистрибутива (замените Debian, если имя другое)
wsl --terminate Debian
```

```powershell
# Удаление дистрибутива
wsl --unregister Debian
```

```powershell
# Проверка, что дистрибутив удален
wsl --list
```

```powershell
# Если вы устанавливали вручную (Вариант 2.2), не забудьте удалить папку (`C:\WSL\Debian` в примере).
# Remove-Item -Recurse -Force C:\WSL\Debian
```

## ❗ Важные примечания

1.  **Драйверы NVIDIA**: Устанавливаются **только в Windows**. Они обеспечивают базовую поддержку CUDA для WSL.
2.  **CUDA Toolkit (в Debian)**: Устанавливается в WSL (как описано в разделе 5.2, **используя репозиторий NVIDIA**) для получения компилятора (`nvcc`) и библиотек. **Требует настройки переменных окружения** (Раздел 5.3).
3.  **NVIDIA Container Toolkit (в Debian)**: Устанавливается в WSL (Раздел 5.1) для того, чтобы **Docker-контейнеры** могли использовать GPU.
4.  **Группы пользователя**: Группы `audio`, `video` обычно **не требуются**. Группа `docker` нужна для запуска команд docker без `sudo`.
5.  **WSLg**: Для GUI-приложений убедитесь, что WSL обновлен (`wsl --update`).
6.  **Пакеты для разработки**: `build-essential` устанавливает базовый набор. `cuda-toolkit` устанавливает инструменты для CUDA-разработки.

## 🔗 Актуальные ссылки

* [Документация Microsoft по WSL](https://learn.microsoft.com/en-us/windows/wsl/)
* [NVIDIA CUDA в WSL (Официальная документация)](https://docs.nvidia.com/cuda/wsl-user-guide/index.html)
* [Загрузка NVIDIA CUDA Toolkit для WSL-Ubuntu (Используется для Debian)](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=WSL-Ubuntu&target_version=2.0&target_type=deb_network)
* [Документация NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)
