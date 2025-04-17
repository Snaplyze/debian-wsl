**📝 Итоговая инструкция по установке и настройке Debian на WSL (Версия 5 - `.run` CUDA, Копипаст-френдли, 17 Апреля 2025)**
*Для Windows 11, Intel Core i7 13700K, NVIDIA RTX 4090*
*Инструкция отформатирована для простого копирования команд по одной строке.*

---

### **1. Подготовка Windows 11**

*(Выполнять в Windows PowerShell от имени администратора)*

#### **1.1 Включение компонентов**

1.  **Виртуализация**:
    * В BIOS/UEFI: Активируйте технологию виртуализации (например, `Intel Virtualization Technology (VT-x)` или `AMD-V`).
    * В Windows: Откройте `Win + R` → `optionalfeatures`. Убедитесь, что включены:
        * `Платформа виртуальной машины (Virtual Machine Platform)`
        * `Подсистема Windows для Linux (Windows Subsystem for Linux)`

2.  **Обновление WSL**:
    ```powershell
    wsl --update
    wsl --set-default-version 2
    ```

---

### **2. Установка Debian WSL**

*(Выполнять в Windows PowerShell)*

#### **2.1 Вариант 1: Установка из Microsoft Store (Простой)**

1.  Откройте Microsoft Store, найдите "Debian".
2.  Нажмите "Получить" и дождитесь установки.
3.  Запустите Debian из меню "Пуск".
4.  При первом запуске следуйте инструкциям для создания пользователя и пароля.

#### **2.2 Вариант 2: Ручная установка с выбором расположения**

```powershell
# Шаг 1: Установка Debian из Microsoft Store без запуска (если еще не установлен)
wsl --install debian --no-launch
# Шаг 2: Создание папки для хранения дистрибутива (пример)
mkdir -p C:\WSL\Debian
# Шаг 3: Экспорт установленного образа в tar-файл
wsl --export Debian C:\WSL\debian-backup.tar
# Шаг 4: Удаление исходного дистрибутива (установленного по умолчанию)
wsl --unregister Debian
# Шаг 5: Импорт в выбранное местоположение
wsl --import Debian C:\WSL\Debian C:\WSL\debian-backup.tar --version 2
# Шаг 6: Удаление временного tar-файла
Remove-Item C:\WSL\debian-backup.tar
# Шаг 7: Запустите Debian (имя дистрибутива 'Debian' как в --import)
wsl -d Debian
# Примечание: При первом запуске после ручного импорта может потребоваться настроить пользователя (см. Раздел 4).
```

#### **2.3 Изменение местоположения уже установленного дистрибутива**

```powershell
# Остановите дистрибутив (замените Debian на ваше имя, если отличается)
wsl --terminate Debian
# Экспортируйте его в tar-файл (пример пути)
wsl --export Debian C:\temp\debian-backup.tar
# Удалите исходный дистрибутив
wsl --unregister Debian
# Импортируйте его в новое местоположение (пример пути)
wsl --import Debian C:\WSL\Debian C:\temp\debian-backup.tar --version 2
# Можно удалить временный tar-файл
Remove-Item C:\temp\debian-backup.tar
```

#### **2.4 Запуск Debian WSL**

```powershell
# Запуск Debian (имя дистрибутива 'Debian' или ваше)
wsl -d Debian
```

---

### **3. Базовая настройка Debian**

*(Выполнять в терминале Debian WSL)*

#### **3.1 Обновление репозиториев и системы**

```bash
# Обновление списка пакетов
sudo apt update
# Обновление установленных пакетов
sudo apt upgrade -y
# Установка базовых пакетов для работы с репозиториями и сетью
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release wget
# Включение contrib, non-free и non-free-firmware репозиториев
sudo sed -i 's/main/main contrib non-free non-free-firmware/g' /etc/apt/sources.list
# Обновление списка пакетов после добавления репозиториев
sudo apt update
```

#### **3.2 Локализация и время**

```bash
# Установка пакетов локализации и часовых поясов
sudo apt install -y locales tzdata
# Раскомментирование нужных локалей в файле конфигурации
sudo sed -i 's/# ru_RU.UTF-8 UTF-8/ru_RU.UTF-8 UTF-8/' /etc/locale.gen
sudo sed -i 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen
# Генерация локалей
sudo locale-gen
# Установка системной локали по умолчанию
sudo update-locale LANG=ru_RU.UTF-8
# Настройка глобальных переменных окружения (перезапись файла)
echo "LANG=ru_RU.UTF-8" | sudo tee /etc/environment
# Добавление переменных в файл
echo "LC_ALL=ru_RU.UTF-8" | sudo tee -a /etc/environment
echo "LANGUAGE=ru_RU:ru" | sudo tee -a /etc/environment
# Установка часового пояса (пример: Москва)
# Если вы не в Москве, найдите свой пояс: timedatectl list-timezones
sudo timedatectl set-timezone Europe/Moscow
```

#### **3.3 Настройка WSL (`/etc/wsl.conf` и `.wslconfig`)**

1.  **Создание/Редактирование файла `/etc/wsl.conf` внутри Debian:**
    *(Выполнять в терминале Debian WSL)*

    ```bash
    # Создание/перезапись файла /etc/wsl.conf
    echo "[user]" | sudo tee /etc/wsl.conf
    # Добавление строк (используйте '-a' для добавления)
    echo "default=wsluser" | sudo tee -a /etc/wsl.conf
    echo "" | sudo tee -a /etc/wsl.conf
    echo "[interop]" | sudo tee -a /etc/wsl.conf
    echo "enabled=true" | sudo tee -a /etc/wsl.conf
    echo "appendWindowsPath=true" | sudo tee -a /etc/wsl.conf
    echo "" | sudo tee -a /etc/wsl.conf
    echo "[boot]" | sudo tee -a /etc/wsl.conf
    echo "systemd=true" | sudo tee -a /etc/wsl.conf
    echo "" | sudo tee -a /etc/wsl.conf
    echo "[network]" | sudo tee -a /etc/wsl.conf
    echo "generateResolvConf = true" | sudo tee -a /etc/wsl.conf
    ```
    *Примечание: После изменения `/etc/wsl.conf` нужен перезапуск WSL (`wsl --shutdown` в PowerShell).*

2.  **Создание/Изменение файла `.wslconfig` на Windows:**
    *(Выполнять на Windows)*
    * Создайте или отредактируйте **текстовый** файл `C:\Users\<Ваше_Имя_Пользователя>\.wslconfig` (замените `<Ваше_Имя_Пользователя>`).
    * Добавьте в него следующее примерное содержимое (или измените существующее):

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
      ```
    * *Примечание: После создания/изменения `.wslconfig` нужен перезапуск WSL (`wsl --shutdown` в PowerShell).*

---

### **4. Создание пользователя**

*(Выполнять в терминале Debian WSL)*

```bash
# Установка fish shell и sudo
sudo apt install -y fish sudo
# Установка fish shell по умолчанию для root
sudo chsh -s /usr/bin/fish root

# --- Создание пользователя wsluser ---
# Сначала проверьте, существует ли пользователь 'wsluser'
id wsluser

# Если предыдущая команда выдала ошибку "no such user":
#   Создаем пользователя
#   sudo useradd -m -G sudo -s /usr/bin/fish wsluser
#   Устанавливаем пароль для нового пользователя (следуйте инструкциям)
#   sudo passwd wsluser

# Если предыдущая команда показала информацию о пользователе:
#   Пользователь уже существует, просто убедимся, что он в группе sudo и у него fish
#   sudo usermod -aG sudo wsluser
#   sudo usermod -s /usr/bin/fish wsluser

# --- Настройка sudo без пароля для wsluser (удобно, но менее безопасно) ---
echo "wsluser ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/wsluser
sudo chmod 440 /etc/sudoers.d/wsluser
```
*Примечание: Раскомментируйте и выполните команды для создания пользователя или изменения существующего в зависимости от вывода команды `id wsluser`. Убедитесь, что `default=wsluser` указан в `/etc/wsl.conf`.*

---

### **5. Установка компонентов NVIDIA**

*(Выполнять в терминале Debian WSL)*

* **❗ Важно:** Убедитесь, что на **Windows** установлен **последний драйвер NVIDIA**.

#### **5.1 Установка NVIDIA Container Toolkit (для Docker GPU)**

```bash
# Добавление GPG-ключа NVIDIA Container Toolkit
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
# Добавление репозитория NVIDIA Container Toolkit
curl -fsSL https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | sudo sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
# Обновление списка пакетов
sudo apt update
# Установка NVIDIA Container Toolkit
sudo apt install -y nvidia-container-toolkit
```

#### **5.2 Установка CUDA Toolkit (Инструменты разработчика) через `.run` файл**

* **⚠️ ПРЕДУПРЕЖДЕНИЕ О `.run` файле в WSL:**
    * Этот метод использует универсальный установщик для Linux.
    * В процессе установки вам будет предложено выбрать компоненты. **ОБЯЗАТЕЛЬНО СНИМИТЕ/ОТКЛЮЧИТЕ галочку напротив компонента "Driver"**. Установка драйвера из `.run` файла в WSL приведет к неработоспособности CUDA.
    * Выберите для установки "CUDA Toolkit" и, по желанию, "CUDA Samples".
    * Метод с `.deb` пакетом (из предыдущей версии инструкции) может быть проще и безопаснее для WSL, так как он интегрирован с `apt` и не пытается установить драйвер.

```bash
# Переходим в домашнюю директорию (или другую временную папку)
cd ~
# Скачиваем .run установщик CUDA (Пример для 12.4.1 - ПРОВЕРЬТЕ АКТУАЛЬНУЮ ВЕРСИЮ/ССЫЛКУ на сайте NVIDIA!)
# Найдите нужный .run файл здесь: https://developer.nvidia.com/cuda-downloads (выбрав Linux -> x86_64 -> Runfile)
wget https://developer.download.nvidia.com/compute/cuda/12.4.1/local_installers/cuda_12.4.1_550.54.15_linux.run

# Запускаем установщик с правами sudo. Потребуется ваше взаимодействие!
sudo sh cuda_12.4.1_550.54.15_linux.run

# --- Следуйте инструкциям на экране ---
# 1. Прочитайте и примите лицензионное соглашение (EULA).
# 2. На экране выбора компонентов:
#    - Снимите выделение (клавишей 'пробел' или 'enter') с компонента [X] Driver
#    - Убедитесь, что выделены компоненты [X] CUDA Toolkit 12.4 (или ваша версия)
#    - По желанию выделите [X] CUDA Samples 12.4
# 3. Выберите "Install" и дождитесь завершения.
# 4. Возможно, потребуется подтвердить пути установки (обычно /usr/local/cuda-12.4).

# Очистка (удаление скачанного .run файла)
rm cuda_12.4.1_550.54.15_linux.run
```

#### **5.3 Настройка переменных окружения CUDA для Fish Shell**

*(Этот шаг **необходим** и не зависит от метода установки Toolkit)*

```bash
# --- Для пользователя wsluser ---
# Добавляем пути в config.fish пользователя (выполнять от имени wsluser, НЕ под sudo)
echo '# Add CUDA paths to environment' >> ~/.config/fish/config.fish
echo 'if test -d /usr/local/cuda' >> ~/.config/fish/config.fish
# Используем 'set -xp ... $PATH' для добавления в начало PATH/LD_LIBRARY_PATH в fish
echo '    set -xp PATH /usr/local/cuda/bin $PATH' >> ~/.config/fish/config.fish
echo '    set -xp LD_LIBRARY_PATH /usr/local/cuda/lib64 $LD_LIBRARY_PATH' >> ~/.config/fish/config.fish
echo 'end' >> ~/.config/fish/config.fish

# --- Для пользователя root ---
# Добавляем пути в config.fish пользователя root
echo '' | sudo tee -a /root/.config/fish/config.fish
echo '# Add CUDA paths to environment' | sudo tee -a /root/.config/fish/config.fish
echo 'if test -d /usr/local/cuda' | sudo tee -a /root/.config/fish/config.fish
# Используем одинарные кавычки вокруг строки для echo, чтобы $PATH не раскрылся сразу
echo '    set -xp PATH /usr/local/cuda/bin $PATH' | sudo tee -a /root/.config/fish/config.fish
echo '    set -xp LD_LIBRARY_PATH /usr/local/cuda/lib64 $LD_LIBRARY_PATH' | sudo tee -a /root/.config/fish/config.fish
echo 'end' | sudo tee -a /root/.config/fish/config.fish
```
*Примечание: Перезапустите WSL или сессию Fish, чтобы переменные применились.*

---

### **6. Установка ПО**

*(Выполнять в терминале Debian WSL от имени пользователя wsluser)*

#### **6.1 Системные утилиты**

```bash
# Установка базовых утилит одной строкой
sudo apt install -y nano python3 python3-pip python3-venv htop curl wget unzip git build-essential pkg-config fzf fd-find bat
```

#### **6.2 Fish Shell с популярными плагинами**

```bash
# Создание директорий для конфигурации пользователя (если еще не созданы)
mkdir -p ~/.config/fish/functions
mkdir -p ~/.config/fish/completions

# --- Настройка fish для пользователя wsluser ---
# Добавляем настройки в ~/.config/fish/config.fish (создастся, если нет)
# Используйте echo ... >> чтобы добавлять строки в конец файла
echo '' >> ~/.config/fish/config.fish
echo '# Алиасы' >> ~/.config/fish/config.fish
echo "alias ll='ls -la'" >> ~/.config/fish/config.fish
echo "alias la='ls -A'" >> ~/.config/fish/config.fish
echo "alias l='ls'" >> ~/.config/fish/config.fish
echo "alias cls='clear'" >> ~/.config/fish/config.fish
echo "alias ..='cd ..'" >> ~/.config/fish/config.fish
echo "alias ...='cd ../..'" >> ~/.config/fish/config.fish
echo '' >> ~/.config/fish/config.fish
echo '# Улучшенные утилиты (bat, fd)' >> ~/.config/fish/config.fish
# Осторожно с кавычками внутри! Используем одинарные кавычки в alias.
echo "type -q batcat && alias cat='batcat --paging=never'" >> ~/.config/fish/config.fish
echo "type -q fdfind && alias fd='fdfind'" >> ~/.config/fish/config.fish
echo "type -q fdfind && alias find='fdfind'" >> ~/.config/fish/config.fish
echo '' >> ~/.config/fish/config.fish
echo '# Настройка fish' >> ~/.config/fish/config.fish
# Отключаем стандартное приветствие через универсальную переменную
echo 'set -U fish_greeting ""' >> ~/.config/fish/config.fish
echo 'set fish_key_bindings fish_default_key_bindings' >> ~/.config/fish/config.fish
echo 'set fish_autosuggestion_enabled 1' >> ~/.config/fish/config.fish
echo '' >> ~/.config/fish/config.fish
echo '# FZF интеграция' >> ~/.config/fish/config.fish
# Осторожно с кавычками и переменными! Используем одинарные кавычки для echo.
echo 'set -gx FZF_DEFAULT_COMMAND "fdfind --type f --color=always --strip-cwd-prefix 2>/dev/null || find . -type f"' >> ~/.config/fish/config.fish
echo 'set -gx FZF_CTRL_T_COMMAND $FZF_DEFAULT_COMMAND' >> ~/.config/fish/config.fish
echo 'set -gx FZF_CTRL_T_OPTS "--preview '\''batcat --color=always --style=numbers --line-range :500 {}'\''"' >> ~/.config/fish/config.fish
echo 'set -gx FZF_ALT_C_COMMAND "fdfind --type d --color=always --strip-cwd-prefix 2>/dev/null || find . -type d"' >> ~/.config/fish/config.fish
echo 'set -gx FZF_ALT_C_OPTS "--preview '\''tree -C {} | head -n 50'\''"' >> ~/.config/fish/config.fish

# Создание функции приветствия для пользователя (перезапись файла)
echo 'function fish_greeting' > ~/.config/fish/functions/fish_greeting.fish
# Добавление строк в функцию
echo '    set -l date_str (date "+%Y-%m-%d %H:%M")' >> ~/.config/fish/functions/fish_greeting.fish
echo '    echo "🐧 WSL Debian User - $date_str"' >> ~/.config/fish/functions/fish_greeting.fish
echo 'end' >> ~/.config/fish/functions/fish_greeting.fish

# Скачивание автозавершений для Docker
curl -sL https://raw.githubusercontent.com/docker/cli/master/contrib/completion/fish/docker.fish -o ~/.config/fish/completions/docker.fish
curl -sL https://raw.githubusercontent.com/docker/compose/master/contrib/completion/fish/docker-compose.fish -o ~/.config/fish/completions/docker-compose.fish

# Установка менеджера плагинов Fisher
fish -c "curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher"
# Установка плагинов через Fisher
fish -c "fisher install jethrokuan/z"
fish -c "fisher install PatrickF1/fzf.fish"
fish -c "fisher install jorgebucaran/autopair.fish"
fish -c "fisher install franciscolourenco/done"
fish -c "fisher install edc/bass"

# Установка Starship prompt
curl -sS https://starship.rs/install.sh | sh -s -- -y
# Добавление инициализации Starship в конфиг fish
echo 'starship init fish | source' >> ~/.config/fish/config.fish

# --- Настройка fish для root ---
# Создание директорий для root (если еще не созданы)
sudo mkdir -p /root/.config/fish/functions
sudo mkdir -p /root/.config/fish/completions

# Добавляем настройки в /root/.config/fish/config.fish
echo '' | sudo tee -a /root/.config/fish/config.fish
echo '# Алиасы' | sudo tee -a /root/.config/fish/config.fish
echo "alias ll='ls -la'" | sudo tee -a /root/.config/fish/config.fish
echo "alias la='ls -A'" | sudo tee -a /root/.config/fish/config.fish
echo "alias l='ls'" | sudo tee -a /root/.config/fish/config.fish
echo "alias cls='clear'" | sudo tee -a /root/.config/fish/config.fish
echo "alias ..='cd ..'" | sudo tee -a /root/.config/fish/config.fish
echo "alias ...='cd ../..'" | sudo tee -a /root/.config/fish/config.fish
echo '' | sudo tee -a /root/.config/fish/config.fish
echo '# Улучшенные утилиты (bat, fd)' | sudo tee -a /root/.config/fish/config.fish
echo "type -q batcat && alias cat='batcat --paging=never'" | sudo tee -a /root/.config/fish/config.fish
echo "type -q fdfind && alias fd='fdfind'" | sudo tee -a /root/.config/fish/config.fish
echo "type -q fdfind && alias find='fdfind'" | sudo tee -a /root/.config/fish/config.fish
echo '' | sudo tee -a /root/.config/fish/config.fish
echo '# Настройка fish' | sudo tee -a /root/.config/fish/config.fish
echo 'set -U fish_greeting ""' | sudo tee -a /root/.config/fish/config.fish
echo 'set fish_key_bindings fish_default_key_bindings' | sudo tee -a /root/.config/fish/config.fish
echo 'set fish_autosuggestion_enabled 1' | sudo tee -a /root/.config/fish/config.fish
echo '' | sudo tee -a /root/.config/fish/config.fish
echo '# FZF интеграция' | sudo tee -a /root/.config/fish/config.fish
# Используем одинарные кавычки для echo, чтобы сохранить внутренние кавычки и переменные
echo 'set -gx FZF_DEFAULT_COMMAND "fdfind --type f --color=always --strip-cwd-prefix 2>/dev/null || find . -type f"' | sudo tee -a /root/.config/fish/config.fish
echo 'set -gx FZF_CTRL_T_COMMAND $FZF_DEFAULT_COMMAND' | sudo tee -a /root/.config/fish/config.fish
echo 'set -gx FZF_CTRL_T_OPTS "--preview '\''batcat --color=always --style=numbers --line-range :500 {}'\''"' | sudo tee -a /root/.config/fish/config.fish
echo 'set -gx FZF_ALT_C_COMMAND "fdfind --type d --color=always --strip-cwd-prefix 2>/dev/null || find . -type d"' | sudo tee -a /root/.config/fish/config.fish
echo 'set -gx FZF_ALT_C_OPTS "--preview '\''tree -C {} | head -n 50'\''"' | sudo tee -a /root/.config/fish/config.fish
echo '' | sudo tee -a /root/.config/fish/config.fish
# Установка Starship (если еще не установлен) и добавление в конфиг root
sudo sh -c 'curl -sS https://starship.rs/install.sh | sh -s -- -y'
echo 'starship init fish | source' | sudo tee -a /root/.config/fish/config.fish

# Создание функции приветствия для root (перезапись файла)
echo 'function fish_greeting' | sudo tee /root/.config/fish/functions/fish_greeting.fish
# Добавление строк в функцию (используем одинарные кавычки для echo)
echo '    set -l date_str (date "+%Y-%m-%d %H:%M")' | sudo tee -a /root/.config/fish/functions/fish_greeting.fish
echo '    echo "🔥 WSL Debian [ROOT] - $date_str"' | sudo tee -a /root/.config/fish/functions/fish_greeting.fish
echo 'end' | sudo tee -a /root/.config/fish/functions/fish_greeting.fish

# Копирование автозавершений Docker для root (игнорируем ошибку, если файла нет)
sudo cp ~/.config/fish/completions/docker.fish /root/.config/fish/completions/ 2>/dev/null || true
sudo cp ~/.config/fish/completions/docker-compose.fish /root/.config/fish/completions/ 2>/dev/null || true
```

#### **6.3 Docker с поддержкой GPU**

*(Выполнять в терминале Debian WSL)*

```bash
# Добавление GPG-ключа Docker
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
# Добавление репозитория Docker
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# Обновление списка пакетов
sudo apt update
# Установка Docker
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
# Настройка Docker для использования NVIDIA GPU (требует nvidia-container-toolkit из раздела 5.1)
sudo nvidia-ctk runtime configure --runtime=docker
# Перезапуск Docker для применения настроек runtime (если он уже был запущен)
# sudo systemctl restart docker # Можно выполнить позже или при старте системы
# Добавление вашего пользователя в группу docker (чтобы не использовать sudo)
sudo usermod -aG docker wsluser
# Включение автозапуска Docker через systemd
sudo systemctl enable docker
# Запуск службы Docker (если не запущена)
sudo systemctl start docker
# Проверка статуса Docker
sudo systemctl status docker
```
*Примечание: После добавления пользователя в группу `docker` **необходимо выйти из сессии WSL и войти снова** (или перезапустить WSL: `wsl --shutdown`), чтобы использовать `docker` без `sudo`.*

---

### **7. Проверка системы**

*(Выполнять в терминале Debian WSL от имени пользователя wsluser)*
* **Важно:** Перед проверкой перезапустите WSL (`wsl --shutdown` в PowerShell, затем `wsl -d Debian`) или хотя бы выйдите и снова войдите в сессию Debian WSL, чтобы применились все настройки групп и переменных окружения.

#### **7.1 NVIDIA и CUDA**

```bash
# 1. Проверка доступности NVIDIA GPU (должна показать информацию о карте)
nvidia-smi
# 2. Проверка версии CUDA Toolkit (nvcc) (должна показать версию)
nvcc --version
# 3. Проверка Docker с NVIDIA GPU (должна вывести то же, что nvidia-smi)
# Убедитесь, что Docker запущен: sudo systemctl status docker
docker run --rm --gpus all nvidia/cuda:latest-base nvidia-smi
```
*Если `nvcc --version` не работает, вернитесь к разделу 5.3 и проверьте переменные окружения. Если `docker run ...` требует `sudo`, выйдите и войдите снова.*

#### **7.2 Локализация и время**

```bash
# Проверка настроек локализации (ожидается ru_RU.UTF-8)
locale
# Проверка часового пояса (ожидается Europe/Moscow или ваш)
timedatectl | grep "Time zone"
```

#### **7.3 Fish и Docker**

```bash
# Проверка текущей оболочки (ожидается /usr/bin/fish)
echo $SHELL
# Проверка работы Docker без sudo (должна вывести пустой список или запущенные контейнеры)
docker ps
```
*Проверка автозавершения Fish: Введите `docker ` (с пробелом) и нажмите `Tab`. Введите `docker run --gpus ` (с пробелом) и нажмите `Tab`.*

---

### **8. Финализация**

*(Выполнять в терминале Debian WSL)*

```bash
# Финальное обновление системы
sudo apt update
sudo apt upgrade -y
# Удаление ненужных пакетов
sudo apt autoremove -y
# Очистка кеша apt
sudo apt clean
# Сообщение о завершении
echo "Настройка Debian WSL завершена!"
```
*Рекомендуется еще раз перезапустить WSL (`wsl --shutdown` в PowerShell) для чистоты.*

---

### **9. Устранение типичных проблем**

*(Команды для выполнения в терминале Debian WSL)*

```bash
# Исправление прав доступа к конфигам fish пользователя (если создавали с sudo)
sudo chown -R wsluser:wsluser /home/wsluser/.config/fish
chmod -R u+rwX /home/wsluser/.config/fish
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
    # Убедитесь, что при установке через .run файл НЕ был установлен компонент Driver.
    ```
* **Проблемы с иконкой Debian в Windows Terminal:** Откройте настройки Windows Terminal (Ctrl+,) -> Открыть JSON-файл, найдите профиль Debian и исправьте/удалите неверную строку `"icon"`.

---

### **10. Удаление WSL-дистрибутива**

*(Выполнять в Windows PowerShell от имени администратора)*

```powershell
# Остановка работающего дистрибутива (замените Debian, если имя другое)
wsl --terminate Debian
# Удаление дистрибутива
wsl --unregister Debian
# Проверка, что дистрибутив удален
wsl --list
# Если вы устанавливали вручную (Вариант 2.2), не забудьте удалить папку (`C:\WSL\Debian` в примере).
```

---
*(Разделы "Важные примечания" и "Актуальные ссылки" остаются без изменений, но помните о предупреждении относительно `.run` файла в разделе 5.2)*
