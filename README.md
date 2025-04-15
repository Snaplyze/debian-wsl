Хорошо, я внесу исправления в инструкцию, убрав избыточные шаги и уточнив некоторые моменты, но сохраню подробную настройку Fish Shell для root, как ты просил. Изменения в настройках пользователя (например, удаление дублирующих переменных окружения) будут зеркально отражены и для root.

---

**📝 Итоговая инструкция по установке и настройке Debian на WSL (Версия 2, Апрель 2025)**
*Для Windows 11, Intel Core i7 13700K, NVIDIA RTX 4090*

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
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

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
sudo ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
sudo dpkg-reconfigure -f noninteractive tzdata
```

#### **3.3 Настройка WSL (`/etc/wsl.conf` и `.wslconfig`)**

1.  **Создание файла `/etc/wsl.conf` внутри Debian:**
    * Этот файл управляет поведением конкретного дистрибутива WSL.
    ```bash
    # Создание файла настроек WSL (без дублирования LANG/LC_ALL)
    sudo tee /etc/wsl.conf > /dev/null << EOL
    [user]
    default=wsluser # Убедитесь, что пользователь с таким именем будет создан

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
    memory=16GB             # Например, 16 ГБ ОЗУ
    processors=8            # Например, 8 процессорных ядер
    # swap=4GB                # Можно указать файл подкачки, если нужно

    # Настройки для поддержки GUI приложений (WSLg)
    guiApplications=true

    # Параметры ядра для совместимости (могут быть нужны для systemd/cgroups)
    kernelCommandLine = cgroup_no_v1=all systemd.unified_cgroup_hierarchy=1

    # Отладка (если нужно)
    # debugShell=true
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
sudo useradd -m -G sudo -s /usr/bin/fish wsluser

# Установка пароля для пользователя
sudo passwd wsluser

# Настройка sudo без пароля (удобно, но менее безопасно)
echo "wsluser ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/wsluser
sudo chmod 440 /etc/sudoers.d/wsluser

# Если вы создали пользователя через useradd, установите его как пользователя по умолчанию
# для входа в WSL (если еще не настроено в /etc/wsl.conf или через Microsoft Store).
# Можно сделать это через реестр Windows или убедиться, что настройка в /etc/wsl.conf верна.
```

---

### **5. Установка компонентов NVIDIA**

#### **5.1 Компоненты NVIDIA в Debian**

**Важное примечание**: Убедитесь, что на **Windows** установлен **последний драйвер NVIDIA** для вашей видеокарты (RTX 4090) с [официального сайта](https://www.nvidia.com/Download/index.aspx). Сам драйвер в Debian **не устанавливается**.

```bash
# Добавление официального GPG-ключа NVIDIA
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

# Добавление репозитория NVIDIA Container Toolkit
curl -fsSL https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sudo sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Обновление списка пакетов
sudo apt update

# Установка NVIDIA Container Toolkit (для поддержки GPU в Docker)
sudo apt install -y nvidia-container-toolkit

# Установка инструментов CUDA (опционально, если нужно компилировать/запускать CUDA-код напрямую в WSL)
# Эти пакеты НЕ устанавливают драйвер, только библиотеки и утилиты (nvcc, nvidia-smi и т.д.)
# Вы можете установить полный набор:
sudo apt install -y nvidia-cuda-toolkit
# ИЛИ, если нужен только nvidia-smi и базовые библиотеки, можно поискать пакет nvidia-utils:
# sudo apt search nvidia-utils
# sudo apt install nvidia-utils-XYZ # Замените XYZ на нужную версию

# Проверка установки утилиты nvidia-smi (должна работать после установки toolkit или utils)
nvidia-smi
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
*(Настройки LANG/LC_ALL удалены из конфигов fish, т.к. они заданы глобально)*

```bash
# Создание директорий для конфигурации пользователя
mkdir -p ~/.config/fish/functions
mkdir -p ~/.config/fish/completions

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
echo 'set -U fish_greeting' >> ~/.config/fish/config.fish # Отключаем стандартное приветствие
echo 'set fish_key_bindings fish_default_key_bindings' >> ~/.config/fish/config.fish
echo 'set fish_autosuggestion_enabled 1' >> ~/.config/fish/config.fish
echo '' >> ~/.config/fish/config.fish
echo '# FZF интеграция (требует установки fzf, fd-find)' >> ~/.config/fish/config.fish
echo "set -gx FZF_DEFAULT_COMMAND 'fdfind --type f --strip-cwd-prefix 2>/dev/null || find . -type f'" >> ~/.config/fish/config.fish
echo 'set -gx FZF_CTRL_T_COMMAND $FZF_DEFAULT_COMMAND' >> ~/.config/fish/config.fish

# Создание приветственного сообщения для пользователя
echo 'function fish_greeting' > ~/.config/fish/functions/fish_greeting.fish
echo '    echo "🐧 WSL Debian User - $(date '\''+%Y-%m-%d %H:%M'\'')"' >> ~/.config/fish/functions/fish_greeting.fish
echo 'end' >> ~/.config/fish/functions/fish_greeting.fish

# Настройка завершений для Docker
curl -sL https://raw.githubusercontent.com/docker/cli/master/contrib/completion/fish/docker.fish -o ~/.config/fish/completions/docker.fish
curl -sL https://raw.githubusercontent.com/docker/compose/master/contrib/completion/fish/docker-compose.fish -o ~/.config/fish/completions/docker-compose.fish

# Установка Fisher и плагинов (выполнять от имени пользователя wsluser)
fish -c "curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher"
fish -c "fisher install jethrokuan/z"
fish -c "fisher install PatrickF1/fzf.fish" # Интеграция fzf
fish -c "fisher install jorgebucaran/autopair.fish"
fish -c "fisher install franciscolourenco/done"
fish -c "fisher install edc/bass"

# Установка Starship (выполнять от имени пользователя wsluser)
curl -sS https://starship.rs/install.sh | sh -s -- -y
echo 'starship init fish | source' >> ~/.config/fish/config.fish

# --- Настройка fish для root ---
sudo mkdir -p /root/.config/fish/functions
sudo mkdir -p /root/.config/fish/completions

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
sudo bash -c 'echo "set -U fish_greeting" >> /root/.config/fish/config.fish' # Отключаем стандартное приветствие
sudo bash -c 'echo "set fish_key_bindings fish_default_key_bindings" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "set fish_autosuggestion_enabled 1" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "# FZF интеграция" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "set -gx FZF_DEFAULT_COMMAND '\''fdfind --type f --strip-cwd-prefix 2>/dev/null || find . -type f'\''" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "set -gx FZF_CTRL_T_COMMAND \$FZF_DEFAULT_COMMAND" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "" >> /root/.config/fish/config.fish'
# Установка Starship для root (если еще не установлен глобально) и добавление в конфиг
sudo sh -c 'curl -sS https://starship.rs/install.sh | sh -s -- -y'
sudo bash -c 'echo "starship init fish | source" >> /root/.config/fish/config.fish'

# Приветствие для root
sudo bash -c 'echo "function fish_greeting" > /root/.config/fish/functions/fish_greeting.fish'
sudo bash -c 'echo "    echo \"🔥 WSL Debian [ROOT] - \$(date '\''+%Y-%m-%d %H:%M'\'')\""  >> /root/.config/fish/functions/fish_greeting.fish'
sudo bash -c 'echo "end" >> /root/.config/fish/functions/fish_greeting.fish'

# Копирование автозавершений Docker для root (если Docker используется под root)
sudo cp ~/.config/fish/completions/docker.fish /root/.config/fish/completions/ 2>/dev/null || true
sudo cp ~/.config/fish/completions/docker-compose.fish /root/.config/fish/completions/ 2>/dev/null || true
# Установка Fisher и плагинов для root (если необходимо, обычно не рекомендуется)
# sudo fish -c "curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher"
# sudo fish -c "fisher install ..." # Добавьте нужные плагины
```

#### **6.3 Docker с поддержкой GPU**
```bash
# Установка необходимых пакетов (могут быть уже установлены)
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

# Добавление официального GPG-ключа Docker
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Добавление репозитория Docker
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Установка Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Настройка Docker для использования NVIDIA GPU по умолчанию
# Убедитесь, что nvidia-container-toolkit установлен (см. Раздел 5.1)
sudo nvidia-ctk runtime configure --runtime=docker
# Эта команда должна автоматически настроить /etc/docker/daemon.json
# Проверьте содержимое файла: cat /etc/docker/daemon.json
# Он должен содержать что-то вроде:
# {
#     "default-runtime": "nvidia",
#     "runtimes": {
#         "nvidia": {
#             "args": [],
#             "path": "nvidia-container-runtime"
#         }
#     }
# }
# Если файл не создался или неверный, создайте/исправьте вручную.

# Добавление пользователя в группу docker (чтобы не использовать sudo для docker)
sudo usermod -aG docker wsluser

# Включение и запуск службы Docker через systemd
sudo systemctl enable docker
sudo systemctl start docker

# Проверка статуса Docker
sudo systemctl status docker
```
**Примечание**: После добавления пользователя в группу `docker`, нужно **выйти из сессии WSL и войти снова** (или перезапустить WSL: `wsl --shutdown`), чтобы изменения вступили в силу и `docker` можно было использовать без `sudo`.

---

### **7. Проверка системы**
#### **7.1 NVIDIA и CUDA**
```bash
# Проверка доступности NVIDIA GPU через драйвер Windows (должна работать всегда)
nvidia-smi

# Проверка версии CUDA Toolkit (если установлен nvidia-cuda-toolkit)
nvcc --version

# Проверка Docker с NVIDIA GPU
# Запустите контейнер, использующий GPU (убедитесь, что Docker запущен и настроен)
docker run --rm --gpus all nvidia/cuda:latest-base nvidia-smi
# Если предыдущая команда не сработала без sudo, перезапустите WSL или используйте `newgrp docker`
```

#### **7.2 Локализация и время**
```bash
# Проверка настроек локализации (должно показывать ru_RU.UTF-8)
locale

# Проверка часового пояса (должно показывать Europe/Moscow)
timedatectl | grep "Time zone"
```

#### **7.3 Fish и Docker**
```bash
# Перезапуск WSL для применения всех настроек
# В PowerShell на Windows:
# wsl --shutdown
# wsl -d Debian

# Войдите под пользователем wsluser
# Проверка текущей оболочки (должна быть /usr/bin/fish)
echo $SHELL

# Проверка работы Docker без sudo (после перезапуска WSL или newgrp docker)
docker ps

# Проверка автозавершения Fish для Docker
# Введите 'docker ' и нажмите Tab - должны появиться команды Docker
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

# Рекомендуется еще раз перезапустить WSL
# В PowerShell на Windows:
# wsl --shutdown
```

---

### **9. Устранение типичных проблем**
```bash
# Исправление проблем с правами доступа к конфигам fish (если создавали с sudo)
sudo chown -R wsluser:wsluser /home/wsluser/.config/fish
chmod -R u+rwX /home/wsluser/.config/fish

# Обновление индекса завершений fish (если автодополнение не работает)
fish -c "fish_update_completions"

# Если powerline-шрифты (для Starship) отображаются некорректно в терминале Windows
# Установите шрифт с поддержкой Nerd Font (например, MesloLGS NF)
# Скачайте отсюда: https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf
# Установите его в Windows (ПКМ -> Установить для всех пользователей)
# Выберите этот шрифт в настройках профиля Debian в Windows Terminal

# Проблемы с русской локализацией (если все еще не работает)
# Проверка доступных локалей
locale -a | grep ru_RU.UTF-8
# Убедитесь, что строки в /etc/environment верны и файл сохранен
cat /etc/environment
# Перезапустите WSL

# Проблемы с Windows Terminal (ошибка пути к иконке)
# Откройте настройки Windows Terminal (Ctrl+,) -> Открыть JSON-файл
# Найдите профиль с некорректным значением "icon" и исправьте путь или удалите строку "icon".
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
Remove-Item -Recurse -Force C:\WSL\Debian
# Удаление временного tar-файла (если остался)
Remove-Item C:\WSL\debian-backup.tar
```

---

### **❗ Важные примечания**
*(Без изменений)*
1.  **Драйверы NVIDIA**: Устанавливаются **только в Windows**. В Debian нужны только библиотеки и утилиты (CUDA Toolkit, Container Toolkit).
2.  **Группы пользователя**: Группы `audio`, `video` **не требуются** — доступ к оборудованию управляется через Windows/WSLg.
3.  **WSLg**: Для GUI-приложений убедитесь, что в Windows установлен [драйвер WSLg](https://learn.microsoft.com/en-us/windows/wsl/wslg) (обычно ставится с `wsl --update`).
4.  **Пакеты для разработки**: Используйте `build-essential` для установки базового набора компиляторов и средств разработки.

---

### **🔗 Актуальные ссылки**
*(Без изменений)*
* [Документация Microsoft по WSL](https://learn.microsoft.com/en-us/windows/wsl/)
* [NVIDIA CUDA в WSL](https://docs.nvidia.com/cuda/wsl-user-guide/index.html)
* [Документация Docker с NVIDIA](https://docs.docker.com/config/containers/resource_constraints/#gpu)

---

Надеюсь, эта скорректированная версия инструкции будет более точной и удобной!
