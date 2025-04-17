**📝 Итоговая инструкция по установке и настройке Debian на WSL (Версия 3, 17 Апреля 2025)**
*Для Windows 11, Intel Core i7 13700K, NVIDIA RTX 4090*

---

*(Разделы 1-4 остаются без изменений, как в предоставленной вами инструкции)*

---

### **1. Подготовка Windows 11**
#### **1.1 Включение компонентов**
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

---

### **2. Установка Debian WSL**
*(Этот раздел без изменений, выбирай один из вариантов)*

#### **2.1 Вариант 1: Установка из Microsoft Store**
1.  Откройте Microsoft Store и найдите "Debian"
2.  Нажмите "Получить" и дождитесь завершения установки
3.  После установки нажмите "Запустить" и дождитесь завершения первоначальной настройки
4.  Создайте пользователя и пароль при первом запуске

#### **2.2 Вариант 2: Ручная установка с выбором расположения**
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

#### **2.3 Изменение местоположения уже установленного дистрибутива**
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

#### **2.4 Запуск Debian WSL**
```powershell
# Запуск Debian
wsl -d Debian
```

---

### **3. Базовая настройка Debian**
#### **3.1 Обновление репозиториев и системы**
```bash
# Установка обновлений
sudo apt update
sudo apt upgrade -y

# Установка базовых пакетов для работы с репозиториями и сетью
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release wget

# Включение contrib, non-free и non-free-firmware репозиториев (важно для драйверов и ПО)
sudo sed -i 's/main/main contrib non-free non-free-firmware/g' /etc/apt/sources.list

# Обновление после добавления репозиториев
sudo apt update
```

#### **3.2 Локализация и время**
```bash
# Установка необходимых пакетов (используем locales вместо locales-all)
sudo apt install -y locales tzdata

# Настройка локализации (раскомментируем русскую и английскую UTF-8)
sudo sed -i 's/# ru_RU.UTF-8 UTF-8/ru_RU.UTF-8 UTF-8/' /etc/locale.gen
sudo sed -i 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen
sudo locale-gen

# Настройка системного языка по умолчанию
sudo update-locale LANG=ru_RU.UTF-8

# Настройка глобальных переменных окружения (основной метод)
echo "LANG=ru_RU.UTF-8" | sudo tee /etc/environment
echo "LC_ALL=ru_RU.UTF-8" | sudo tee -a /etc/environment
echo "LANGUAGE=ru_RU:ru" | sudo tee -a /etc/environment

# Установка часового пояса (пример: Москва)
# Если вы не в Москве, найдите свой пояс: timedatectl list-timezones
sudo timedatectl set-timezone Europe/Moscow
# Старый метод (на всякий случай, но timedatectl предпочтительнее):
# sudo ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
# sudo dpkg-reconfigure -f noninteractive tzdata
```

#### **3.3 Настройка WSL (`/etc/wsl.conf` и `.wslconfig`)**

1.  **Создание файла `/etc/wsl.conf` внутри Debian:**
    * Этот файл управляет поведением конкретного дистрибутива WSL.
    ```bash
    # Создание файла настроек WSL
    sudo tee /etc/wsl.conf > /dev/null << EOL
    [user]
    default=wsluser # Убедитесь, что пользователь с таким именем будет создан (см. раздел 4)

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
    * Этот файл (`%USERPROFILE%\.wslconfig`) управляет глобальными настройками WSL2 для *всех* дистрибутивов (память, процессор, GUI и т.д.).
    * Создайте или отредактируйте файл `C:\Users\<Ваше_Имя_Пользователя>\.wslconfig` (замените `<Ваше_Имя_Пользователя>`) со следующим примерным содержимым:
    ```ini
    [wsl2]
    # Настройки ресурсов (примеры, измените под свои нужды)
    memory=16GB          # Например, 16 ГБ ОЗУ
    processors=8         # Например, 8 процессорных ядер
    # swap=4GB             # Можно указать файл подкачки, если нужно

    # Настройки для поддержки GUI приложений (WSLg)
    guiApplications=true

    # Отладка (если нужно)
    # debugShell=true

    # Если возникают проблемы с systemd/cgroups, можно попробовать:
    # kernelCommandLine = cgroup_no_v1=all systemd.unified_cgroup_hierarchy=1
    ```
    * **Примечание**: После создания/изменения `.wslconfig` нужен перезапуск WSL (`wsl --shutdown` в PowerShell).

---

### **4. Создание пользователя**
*(Этот раздел без изменений, т.к. ты хочешь оставить настройку Fish для root)*

```bash
# Установка fish shell и sudo
sudo apt install -y fish sudo

# Установка fish shell по умолчанию для root
sudo chsh -s /usr/bin/fish root

# Создание пользователя wsluser (если не создан при первом запуске)
# Проверьте, существует ли пользователь: id wsluser
# Если нет, создайте:
if ! id "wsluser" &>/dev/null; then
    sudo useradd -m -G sudo -s /usr/bin/fish wsluser
    echo "Пользователь wsluser создан."
    # Установка пароля для пользователя
    echo "Установите пароль для wsluser:"
    sudo passwd wsluser
else
    echo "Пользователь wsluser уже существует."
    # Убедимся, что он в группе sudo и у него правильная оболочка
    sudo usermod -aG sudo wsluser
    sudo usermod -s /usr/bin/fish wsluser
fi


# Настройка sudo без пароля (удобно, но менее безопасно)
echo "wsluser ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/wsluser
sudo chmod 440 /etc/sudoers.d/wsluser

# Убедитесь, что пользователь wsluser установлен как default в /etc/wsl.conf (сделано в разделе 3.3)
```

---

### **5. Установка компонентов NVIDIA**

#### **5.1 Компоненты NVIDIA в Debian**

**❗ Важное примечание**:
* Убедитесь, что на **Windows** установлен **последний драйвер NVIDIA** для вашей видеокарты (RTX 4090) с [официального сайта](https://www.nvidia.com/Download/index.aspx). Сам драйвер в Debian **не устанавливается**. Драйвер Windows передает API CUDA в WSL2.
* Этот раздел устанавливает **две** основные вещи:
    1.  **NVIDIA Container Toolkit**: Позволяет Docker-контейнерам получать доступ к GPU.
    2.  **NVIDIA CUDA Toolkit**: Предоставляет компилятор `nvcc` и библиотеки CUDA для разработки и запуска приложений непосредственно в WSL, а не только в контейнерах.

**1. Установка NVIDIA Container Toolkit (для Docker GPU)**

```bash
# Добавление официального GPG-ключа NVIDIA Container Toolkit
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

# Добавление репозитория NVIDIA Container Toolkit
curl -fsSL https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sudo sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Обновление списка пакетов
sudo apt update

# Установка NVIDIA Container Toolkit
sudo apt install -y nvidia-container-toolkit
```

**2. Установка CUDA Toolkit (по официальной инструкции NVIDIA для WSL)**

* **Примечание:** Команды ниже соответствуют установке CUDA Toolkit 12.4 (на момент Апреля 2025). **Всегда проверяйте актуальные команды** на [официальной странице загрузки CUDA для WSL-Ubuntu](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=WSL-Ubuntu&target_version=2.0&target_type=deb_network), так как версии и URL могут меняться.

```bash
# Скачиваем пиннинг-файл для репозитория CUDA (указывает приоритет)
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600

# Скачиваем deb-пакет репозитория CUDA (пример для CUDA 12.4.1, проверьте актуальную версию!)
# Замените '12.4.1' и '12-4' на актуальные версии при необходимости
CUDA_VERSION_MAJOR_MINOR="12.4"
CUDA_VERSION_FULL="12.4.1"
CUDA_REPO_PACKAGE="cuda-repo-wsl-ubuntu-${CUDA_VERSION_MAJOR_MINOR//./-}-local_${CUDA_VERSION_FULL}-1_amd64.deb" # Обратите внимание, может быть local или network
wget "https://developer.download.nvidia.com/compute/cuda/${CUDA_VERSION_FULL}/local_installers/${CUDA_REPO_PACKAGE}"

# Устанавливаем скачанный пакет репозитория
sudo dpkg -i ${CUDA_REPO_PACKAGE}

# Копируем ключ репозитория (путь может зависеть от версии в имени пакета)
# Убедитесь, что путь /var/cuda-repo-wsl-ubuntu-12-4-local/ совпадает с версией выше
sudo cp "/var/cuda-repo-wsl-ubuntu-${CUDA_VERSION_MAJOR_MINOR//./-}-local/cuda-"*"-keyring.gpg" /usr/share/keyrings/

# Обновляем список пакетов после добавления репозитория CUDA
sudo apt update

# Устанавливаем CUDA Toolkit (мета-пакет, который установит последнюю версию из репозитория)
# Можно указать конкретную версию, например: sudo apt install cuda-toolkit-12-4
sudo apt install -y cuda-toolkit

# Очистка (удаление скачанного deb-файла репозитория)
rm ${CUDA_REPO_PACKAGE}
```

**3. Настройка переменных окружения для CUDA**

* Чтобы система и компиляторы могли найти исполняемые файлы CUDA (как `nvcc`) и библиотеки, нужно добавить пути в переменные окружения $PATH и $LD_LIBRARY_PATH.

```bash
# --- Для пользователя wsluser ---
# Добавляем пути в config.fish пользователя
# Используем /usr/local/cuda как стандартный путь (обычно это символическая ссылка на конкретную версию)
echo '# Add CUDA paths to environment' >> ~/.config/fish/config.fish
echo 'if test -d /usr/local/cuda' >> ~/.config/fish/config.fish
echo '    set -xp PATH /usr/local/cuda/bin $PATH' >> ~/.config/fish/config.fish
echo '    set -xp LD_LIBRARY_PATH /usr/local/cuda/lib64 $LD_LIBRARY_PATH' >> ~/.config/fish/config.fish
echo 'end' >> ~/.config/fish/config.fish

# --- Для root ---
# Добавляем пути в config.fish пользователя root
sudo bash -c 'echo "" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "# Add CUDA paths to environment" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "if test -d /usr/local/cuda" >> /root/.config/fish/config.fish'
sudo bash -c 'echo '\''    set -xp PATH /usr/local/cuda/bin $PATH'\'' >> /root/.config/fish/config.fish'
sudo bash -c 'echo '\''    set -xp LD_LIBRARY_PATH /usr/local/cuda/lib64 $LD_LIBRARY_PATH'\'' >> /root/.config/fish/config.fish'
sudo bash -c 'echo "end" >> /root/.config/fish/config.fish'

# Примените изменения переменных окружения, перезапустив оболочку или WSL
# Для немедленного применения в текущей сессии fish (для пользователя):
# source ~/.config/fish/config.fish
# или просто выйдите и войдите снова.
```

---

### **6. Установка ПО**
#### **6.1 Системные утилиты**
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

#### **6.2 Fish Shell с популярными плагинами**
*(Без изменений, кроме добавления настроек CUDA PATH/LD_LIBRARY_PATH в конце раздела 5)*
```bash
# Создание директорий для конфигурации пользователя
mkdir -p ~/.config/fish/functions
mkdir -p ~/.config/fish/completions

# --- Настройка fish для пользователя wsluser ---
# (Предполагается, что файл уже существует и содержит базовые настройки из раздела 5)
# Добавляем алиасы и прочее, если файла еще нет или он пуст
if [ ! -s ~/.config/fish/config.fish ]; then
    echo '# --- Fish Shell Config WSL Debian ---' > ~/.config/fish/config.fish
fi
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
echo 'set -U fish_greeting ""' >> ~/.config/fish/config.fish # Отключаем стандартное приветствие через универсальную переменную
echo 'set fish_key_bindings fish_default_key_bindings' >> ~/.config/fish/config.fish
echo 'set fish_autosuggestion_enabled 1' >> ~/.config/fish/config.fish
echo '' >> ~/.config/fish/config.fish
echo '# FZF интеграция (требует установки fzf, fd-find)' >> ~/.config/fish/config.fish
echo "set -gx FZF_DEFAULT_COMMAND 'fdfind --type f --color=always --strip-cwd-prefix 2>/dev/null || find . -type f'" >> ~/.config/fish/config.fish
echo 'set -gx FZF_CTRL_T_COMMAND $FZF_DEFAULT_COMMAND' >> ~/.config/fish/config.fish
echo 'set -gx FZF_CTRL_T_OPTS "--preview '\''batcat --color=always --style=numbers --line-range :500 {}'\''"' >> ~/.config/fish/config.fish # Preview с bat
echo 'set -gx FZF_ALT_C_COMMAND "fdfind --type d --color=always --strip-cwd-prefix 2>/dev/null || find . -type d"' >> ~/.config/fish/config.fish # Поиск директорий для Alt+C
echo 'set -gx FZF_ALT_C_OPTS "--preview '\''tree -C {} | head -n 50'\''"' >> ~/.config/fish/config.fish # Preview для директорий

# Создание приветственного сообщения для пользователя
echo 'function fish_greeting' > ~/.config/fish/functions/fish_greeting.fish
echo '    set -l date_str (date "+%Y-%m-%d %H:%M")' >> ~/.config/fish/functions/fish_greeting.fish
echo '    echo "🐧 WSL Debian User - $date_str"' >> ~/.config/fish/functions/fish_greeting.fish
echo 'end' >> ~/.config/fish/functions/fish_greeting.fish

# Настройка завершений для Docker
curl -sL https://raw.githubusercontent.com/docker/cli/master/contrib/completion/fish/docker.fish -o ~/.config/fish/completions/docker.fish
curl -sL https://raw.githubusercontent.com/docker/compose/master/contrib/completion/fish/docker-compose.fish -o ~/.config/fish/completions/docker-compose.fish

# Установка Fisher и плагинов (выполнять от имени пользователя wsluser)
fish -c "curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher"
fish -c "fisher install jethrokuan/z" # Перемещение по часто используемым директориям
fish -c "fisher install PatrickF1/fzf.fish" # Интеграция fzf (Ctrl+T, Alt+C, Ctrl+R)
fish -c "fisher install jorgebucaran/autopair.fish" # Автоматическое закрытие скобок/кавычек
fish -c "fisher install franciscolourenco/done" # Уведомления о завершении долгих команд
fish -c "fisher install edc/bass" # Использование bash утилит/скриптов в fish

# Установка Starship (выполнять от имени пользователя wsluser)
curl -sS https://starship.rs/install.sh | sh -s -- -y
echo 'starship init fish | source' >> ~/.config/fish/config.fish

# --- Настройка fish для root ---
sudo mkdir -p /root/.config/fish/functions
sudo mkdir -p /root/.config/fish/completions

# (Предполагается, что файл уже существует и содержит базовые настройки из раздела 5)
# Добавляем алиасы и прочее, если файла еще нет или он пуст
sudo bash -c 'if [ ! -s /root/.config/fish/config.fish ]; then echo "# --- Fish Shell Config WSL Debian [ROOT] ---" > /root/.config/fish/config.fish; fi'
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
sudo bash -c 'echo "# FZF интеграция" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "set -gx FZF_DEFAULT_COMMAND '\''fdfind --type f --color=always --strip-cwd-prefix 2>/dev/null || find . -type f'\''" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "set -gx FZF_CTRL_T_COMMAND \$FZF_DEFAULT_COMMAND" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "set -gx FZF_CTRL_T_OPTS \"--preview '\''batcat --color=always --style=numbers --line-range :500 {}'\''\"" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "set -gx FZF_ALT_C_COMMAND \"fdfind --type d --color=always --strip-cwd-prefix 2>/dev/null || find . -type d\"" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "set -gx FZF_ALT_C_OPTS \"--preview '\''tree -C {} | head -n 50'\''\"" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "" >> /root/.config/fish/config.fish'
# Установка Starship для root (если еще не установлен глобально) и добавление в конфиг
sudo sh -c 'curl -sS https://starship.rs/install.sh | sh -s -- -y' # Ставим глобально, если еще нет
sudo bash -c 'echo "starship init fish | source" >> /root/.config/fish/config.fish'

# Приветствие для root
sudo bash -c 'echo "function fish_greeting" > /root/.config/fish/functions/fish_greeting.fish'
sudo bash -c 'echo "    set -l date_str (date \"+%Y-%m-%d %H:%M\")" >> /root/.config/fish/functions/fish_greeting.fish'
sudo bash -c 'echo "    echo \"🔥 WSL Debian \[ROOT] - \$date_str\"" >> /root/.config/fish/functions/fish_greeting.fish'
sudo bash -c 'echo "end" >> /root/.config/fish/functions/fish_greeting.fish'

# Копирование автозавершений Docker для root (если Docker используется под root)
sudo cp ~/.config/fish/completions/docker.fish /root/.config/fish/completions/ 2>/dev/null || true
sudo cp ~/.config/fish/completions/docker-compose.fish /root/.config/fish/completions/ 2>/dev/null || true
# Установка Fisher и плагинов для root (обычно не рекомендуется, но если очень нужно)
# sudo fish -c "curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher"
# sudo fish -c "fisher install jethrokuan/z" # Пример
```

#### **6.3 Docker с поддержкой GPU**
```bash
# Установка необходимых пакетов (могут быть уже установлены)
# sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release (уже ставили)

# Добавление официального GPG-ключа Docker
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Добавление репозитория Docker
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Установка Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Настройка Docker для использования NVIDIA GPU по умолчанию
# Убедитесь, что nvidia-container-toolkit установлен (см. Раздел 5.1, шаг 1)
sudo nvidia-ctk runtime configure --runtime=docker
# Эта команда должна автоматически настроить /etc/docker/daemon.json
# Проверьте содержимое файла: cat /etc/docker/daemon.json
# Он должен содержать что-то вроде:
# {
#     "runtimes": {
#         "nvidia": {
#             "args": [],
#             "path": "nvidia-container-runtime"
#         }
#     },
#     "default-runtime": "nvidia" # Это строка может быть добавлена вручную, если не добавилась
# }
# Если файл пуст или не настроен, создайте/исправьте вручную:
# sudo mkdir -p /etc/docker
# sudo tee /etc/docker/daemon.json > /dev/null <<EOF
# {
#     "runtimes": {
#         "nvidia": {
#             "path": "/usr/bin/nvidia-container-runtime",
#             "runtimeArgs": []
#         }
#     },
#     "default-runtime": "nvidia"
# }
# EOF

# Добавление пользователя в группу docker (чтобы не использовать sudo для docker)
sudo usermod -aG docker wsluser

# Включение и запуск службы Docker через systemd (требует systemd=true в /etc/wsl.conf)
sudo systemctl enable docker
sudo systemctl start docker

# Проверка статуса Docker
sudo systemctl status docker
```
**Примечание**: После добавления пользователя в группу `docker`, нужно **выйти из сессии WSL и войти снова** (или перезапустить WSL: `wsl --shutdown`), чтобы изменения вступили в силу и `docker` можно было использовать без `sudo`.

---

### **7. Проверка системы**
#### **7.1 NVIDIA и CUDA**

**Важно:** Перезапустите WSL (`wsl --shutdown` в PowerShell, затем `wsl -d Debian`) или как минимум выйдите и войдите в сессию fish, чтобы применились настройки переменных окружения CUDA из раздела 5.3.

```bash
# 1. Проверка доступности NVIDIA GPU через драйвер Windows (должна работать всегда после установки драйвера в Windows)
nvidia-smi

# 2. Проверка версии CUDA Toolkit (nvcc) (должна работать после установки cuda-toolkit и перезапуска сессии)
nvcc --version
# Если команда не найдена, проверьте:
# - Успешно ли завершилась установка 'cuda-toolkit' в разделе 5.2?
# - Правильно ли добавлены пути в ~/.config/fish/config.fish (раздел 5.3)?
# - Перезапустили ли вы сессию/WSL?
# - Существует ли путь /usr/local/cuda/bin? (ls /usr/local/cuda/bin)

# 3. Проверка Docker с NVIDIA GPU
# Убедитесь, что Docker запущен (sudo systemctl status docker) и настроен (раздел 6.3)
# Запустите тестовый контейнер, использующий GPU. Он должен вывести тот же результат, что и nvidia-smi.
# Запускайте от пользователя wsluser БЕЗ sudo (после перезапуска WSL/сессии)
docker run --rm --gpus all nvidia/cuda:latest-base nvidia-smi
# Если команда 'docker' требует sudo, значит вы не перезапустили сессию после добавления пользователя в группу docker.
# Попробуйте `newgrp docker` в текущей сессии или перезапустите WSL.
```

#### **7.2 Локализация и время**
```bash
# Проверка настроек локализации (должно показывать ru_RU.UTF-8)
locale

# Проверка часового пояса (должно показывать Europe/Moscow или ваш пояс)
timedatectl | grep "Time zone"
```

#### **7.3 Fish и Docker**
```bash
# Проверка текущей оболочки (должна быть /usr/bin/fish)
echo $SHELL

# Проверка работы Docker без sudo (после перезапуска WSL или newgrp docker)
docker ps

# Проверка автозавершения Fish для Docker
# Введите 'docker ' и нажмите Tab - должны появиться команды Docker
# Введите 'docker run --gpus' и нажмите Tab - должны появиться опции --gpus
```

---

### **8. Финализация**
```bash
# Финальное обновление системы
sudo apt update
sudo apt upgrade -y
sudo apt autoremove -y

# Очистка кеша apt
sudo apt clean

echo "Настройка Debian WSL завершена!"

# Рекомендуется еще раз перезапустить WSL для чистоты
# В PowerShell на Windows:
# wsl --shutdown
```

---

### **9. Устранение типичных проблем**
*(Добавлена проверка CUDA путей)*
```bash
# Исправление проблем с правами доступа к конфигам fish (если создавали с sudo)
sudo chown -R wsluser:wsluser /home/wsluser/.config/fish
chmod -R u+rwX /home/wsluser/.config/fish

# Обновление индекса завершений fish (если автодополнение не работает)
fish -c "fish_update_completions"

# Если powerline-шрифты (для Starship) отображаются некорректно в терминале Windows
# Установите шрифт с поддержкой Nerd Font (например, MesloLGS NF, FiraCode NF, Caskaydia Cove NF)
# Скачайте, установите в Windows (ПКМ -> Установить для всех пользователей)
# Выберите этот шрифт в настройках профиля Debian в Windows Terminal (JSON или GUI)

# Проблемы с русской локализацией (если все еще не работает)
# Проверка доступных локалей
locale -a | grep ru_RU.UTF-8
# Убедитесь, что строки в /etc/environment верны и файл сохранен
cat /etc/environment
# Перезапустите WSL

# Проблемы с Windows Terminal (ошибка пути к иконке)
# Откройте настройки Windows Terminal (Ctrl+,) -> Открыть JSON-файл
# Найдите профиль с некорректным значением "icon" и исправьте путь или удалите строку "icon".

# Проблемы с CUDA (`nvcc: command not found` или ошибки библиотек)
# 1. Проверьте переменные окружения в fish:
#    echo $PATH
#    echo $LD_LIBRARY_PATH
#    # Должны содержать /usr/local/cuda/bin и /usr/local/cuda/lib64 соответственно
# 2. Проверьте, куда указывает /usr/local/cuda:
#    ls -l /usr/local/cuda
#    # Должно быть ссылкой на /usr/local/cuda-X.Y
# 3. Убедитесь, что файлы существуют:
#    ls /usr/local/cuda/bin/nvcc
#    ls /usr/local/cuda/lib64/libcudart.so*
# 4. Если пути не установлены, перепроверьте раздел 5.3 и перезапустите сессию.
# 5. Убедитесь, что драйвер NVIDIA в Windows обновлен.
```

---

### **10. Удаление WSL-дистрибутива**
*(Без изменений)*
```powershell
# В PowerShell с правами администратора

# Остановка работающего дистрибутива
wsl --terminate Debian

# Удаление дистрибутива
wsl --unregister Debian

# Проверка, что дистрибутив удален
wsl --list

# Дополнительная очистка (если вы вручную устанавливали дистрибутив)
# Удаление папки дистрибутива (путь из вашего варианта 2.2)
# Remove-Item -Recurse -Force C:\WSL\Debian
# Удаление временного tar-файла (если остался)
# Remove-Item C:\WSL\debian-backup.tar
```

---

### **❗ Важные примечания**
*(Обновлено для ясности)*
1.  **Драйверы NVIDIA**: Устанавливаются **только в Windows**. Они обеспечивают базовую поддержку CUDA для WSL.
2.  **CUDA Toolkit (в Debian)**: Устанавливается в WSL (как описано в разделе 5.2) для получения **компилятора (`nvcc`) и библиотек** для разработки/запуска CUDA-приложений непосредственно в WSL.
3.  **NVIDIA Container Toolkit (в Debian)**: Устанавливается в WSL (как описано в разделе 5.1) для того, чтобы **Docker-контейнеры** могли использовать GPU.
4.  **Группы пользователя**: Группы `audio`, `video` обычно **не требуются** — доступ к оборудованию управляется через Windows/WSLg. Группа `docker` нужна для запуска команд docker без `sudo`.
5.  **WSLg**: Для GUI-приложений убедитесь, что в Windows установлен графический драйвер, поддерживающий WSLg (обычно последние драйверы Intel/AMD/NVIDIA подходят), и что WSL обновлен (`wsl --update`).
6.  **Пакеты для разработки**: `build-essential` устанавливает базовый набор (gcc, g++, make и т.д.). Для CUDA-разработки нужен `cuda-toolkit`.

---

### **🔗 Актуальные ссылки**
*(Добавлена ссылка на загрузку CUDA)*
* [Документация Microsoft по WSL](https://learn.microsoft.com/en-us/windows/wsl/)
* [NVIDIA CUDA в WSL (Официальная документация)](https://docs.nvidia.com/cuda/wsl-user-guide/index.html)
* [Загрузка NVIDIA CUDA Toolkit для WSL-Ubuntu (Используется для Debian)](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=WSL-Ubuntu&target_version=2.0&target_type=deb_network)
* [Документация Docker с NVIDIA](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) (Более актуальная ссылка на Container Toolkit)
