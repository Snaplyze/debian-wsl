# debian-wsl

**📝 Итоговая инструкция по установке и настройке Debian на WSL (актуально на апрель 2025)**  
*Для Windows 11, Intel Core i7 13700K, NVIDIA RTX 4090*

---

### **1. Подготовка Windows 11**  
#### **1.1 Включение компонентов**  
1. **Виртуализация**:  
   - В BIOS/UEFI активируйте:  
     - **Intel Virtualization Technology (VT-x)**.  
     - **Hyper-V** (если доступно).  
   - В Windows:  
     - `Win + R` → `optionalfeatures` → включите:  
       - *Hyper-V*,  
       - *Virtual Machine Platform*,  
       - *Windows Subsystem for Linux*.  

2. **Обновление WSL**:  
   ```powershell
   wsl --update
   wsl --set-default-version 2
   ```

---

### **2. Установка Debian WSL**  
#### **2.1 Вариант 1: Установка из Microsoft Store**  
1. Откройте Microsoft Store и найдите "Debian"
2. Нажмите "Получить" и дождитесь завершения установки
3. После установки нажмите "Запустить" и дождитесь завершения первоначальной настройки
4. Создайте пользователя и пароль при первом запуске

#### **2.1 Вариант 2: Ручная установка**  
```powershell
# Скачайте последнюю версию Debian для WSL
curl.exe -L -o debian-wsl.appx https://aka.ms/wsl-debian-gnulinux

# Распакуйте appx файл (можно сделать переименовав расширение в .zip и распаковав)
Rename-Item .\debian-wsl.appx .\debian-wsl.zip
Expand-Archive .\debian-wsl.zip .\debian-wsl

# Импортируйте дистрибутив
wsl --import Debian C:\WSL\Debian .\debian-wsl\install.tar.gz --version 2
```

#### **2.2 Запуск Debian WSL**  
```powershell
# Запуск Debian (если установлен через Microsoft Store)
wsl -d Debian

# Или если установлен вручную
wsl -d Debian
```

---

### **3. Базовая настройка Debian**  
#### **3.1 Обновление репозиториев и системы**  
```bash
# Установка обновлений
sudo apt update
sudo apt upgrade -y

# Установка базовых пакетов
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

# Включение contrib и non-free репозиториев
sudo sed -i 's/main/main contrib non-free non-free-firmware/g' /etc/apt/sources.list

# Обновление после добавления репозиториев
sudo apt update
```

#### **3.2 Локализация и время**  
```bash
# Установка необходимых пакетов
sudo apt install -y locales locales-all tzdata

# Настройка локализации
sudo sed -i 's/# ru_RU.UTF-8 UTF-8/ru_RU.UTF-8 UTF-8/' /etc/locale.gen
sudo sed -i 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen
sudo locale-gen

# Настройка системного языка
sudo update-locale LANG=ru_RU.UTF-8 LC_ALL=ru_RU.UTF-8

# Настройка глобальных переменных окружения
echo "LANG=ru_RU.UTF-8" | sudo tee -a /etc/environment
echo "LC_ALL=ru_RU.UTF-8" | sudo tee -a /etc/environment
echo "LANGUAGE=ru_RU:ru" | sudo tee -a /etc/environment

# Установка московского времени
sudo ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
sudo dpkg-reconfigure -f noninteractive tzdata
```

#### **3.3 Настройка WSL**  
```bash
# Создание файла настроек WSL
sudo tee /etc/wsl.conf > /dev/null << EOL
[user]
default=wsluser

[interop]
enabled=true
appendWindowsPath=true

[boot]
systemd=true

[environment]
LANG=ru_RU.UTF-8
LC_ALL=ru_RU.UTF-8
EOL
```

---

### **4. Создание пользователя**  
```bash
# Установка необходимых пакетов
sudo apt install -y zsh sudo

# Создание пользователя wsluser
sudo useradd -m -G sudo -s /bin/zsh wsluser

# Установка пароля для пользователя
sudo passwd wsluser

# Настройка sudo без пароля
echo "wsluser ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/wsluser
sudo chmod 440 /etc/sudoers.d/wsluser
```

---

### **5. Установка драйверов и компонентов NVIDIA**  

#### **5.1 Драйверы NVIDIA**  

**Примечание**: На Windows установите последнюю версию драйвера для RTX 4090 с [официального сайта](https://www.nvidia.com/Download/index.aspx).

```bash
# Добавление официального GPG-ключа NVIDIA
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

# Добавление стабильного репозитория NVIDIA (общий для всех deb-based дистрибутивов)
curl -fsSL https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sudo sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Установка утилит и драйверов NVIDIA
sudo apt update
sudo apt install -y nvidia-container-toolkit nvidia-cuda-toolkit cuda-toolkit-12-3
```
  
#### **5.2 Настройка графики (WSLg)**  

**Примечание**: В файл `%USERPROFILE%\.wslconfig` на Windows добавьте:  

```ini
[wsl2]
guiApplications = true
kernelCommandLine = cgroup_no_v1=all systemd.unified_cgroup_hierarchy=1
memory=16GB
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
    htop \
    curl \
    wget \
    unzip \
    git \
    build-essential \
    pkg-config
```

#### **6.2 ZSH + Oh My ZSH с популярными плагинами**  

**Популярные плагины для ZSH:**
1. **zsh-autosuggestions** - предлагает автодополнения на основе истории команд
2. **zsh-syntax-highlighting** - подсветка синтаксиса команд в реальном времени
3. **git** - множество алиасов и функций для работы с Git
4. **docker** - алиасы и автодополнение для Docker
5. **sudo** - добавляет sudo к предыдущей команде при двойном нажатии Esc
6. **z** - умная навигация (быстрые переходы в часто используемые директории)
7. **extract** - универсальная распаковка архивов через команду "x filename"
8. **fzf** - нечеткий поиск файлов, директорий и в истории
9. **history** - улучшенная работа с историей команд
10. **dirhistory** - навигация по истории директорий (Alt+Left/Right/Up/Down)
11. **colored-man-pages** - цветные man-страницы
12. **command-not-found** - предлагает установить пакет, если команда не найдена

```bash
# Установка Oh My ZSH для вашего пользователя (от имени wsluser)
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended

# Установка популярных плагинов
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-completions

# Настройка плагинов в конфигурационном файле
sed -i 's/plugins=(git)/plugins=(git docker zsh-autosuggestions zsh-syntax-highlighting zsh-completions sudo extract z dirhistory colored-man-pages command-not-found)/g' ~/.zshrc

# Настройка автозаполнения
echo '# Улучшенное автозаполнение' >> ~/.zshrc
echo 'autoload -Uz compinit' >> ~/.zshrc
echo 'compinit' >> ~/.zshrc
echo 'zstyle ":completion:*" menu select' >> ~/.zshrc

# Установка правильных прав
chmod -R 755 ~/.oh-my-zsh/custom/plugins
```

#### **6.3 Docker с поддержкой GPU**  
```bash
# Установка необходимых пакетов
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

# Добавление официального GPG-ключа Docker (современный метод)
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Добавление репозитория Docker
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Установка Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Настройка Docker для работы с NVIDIA GPU
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json > /dev/null << EOL
{
  "runtimes": {
    "nvidia": {
      "path": "nvidia-container-runtime",
      "runtimeArgs": []
    }
  },
  "default-runtime": "nvidia"
}
EOL

# Добавление пользователя в группу docker
sudo usermod -aG docker wsluser

# Настройка systemd для Docker
sudo systemctl enable docker
sudo systemctl start docker
```

---

### **7. Проверка системы**  
#### **7.1 NVIDIA и CUDA**  
```bash
# Проверка доступности NVIDIA GPU
nvidia-smi

# Проверка CUDA
nvcc --version

# Проверка Docker с NVIDIA GPU
sudo docker run --rm --gpus all nvidia/cuda:12.3.0-base-ubuntu22.04 nvidia-smi
```

#### **7.2 Локализация и время**  
```bash
# Проверка настроек локализации
locale

# Проверка часового пояса
timedatectl | grep "Time zone"
```

#### **7.3 ZSH и Docker**  
```bash
# Перезапуск WSL для применения всех настроек
# В PowerShell на Windows:
# wsl --terminate Debian
# wsl -d Debian

# Проверка текущей оболочки
echo $SHELL

# Проверка Docker
docker ps
```

---

### **8. Финализация**  
```bash
# Финальное обновление системы
sudo apt update
sudo apt upgrade -y
sudo apt autoremove -y

# Перезапуск WSL для применения всех настроек
# В PowerShell на Windows:
# wsl --terminate Debian
# wsl -d Debian
```

---

### **9. Устранение типичных проблем**  
```bash
# Исправление проблем с автозаполнением в ZSH
compaudit | xargs chmod g-w,o-w

# Обновление индекса автозаполнения
rm -rf ~/.zcompdump*
compinit

# Если powerline-шрифты отображаются некорректно
mkdir -p ~/.local/share/fonts
curl -fLo ~/.local/share/fonts/MesloLGS-NF-Regular.ttf https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf
```

#### **9.1 Проблемы с русской локализацией**
Если русский язык не работает после всех настроек:

```bash
# Проверка доступных локалей
locale -a | grep ru

# Дополнительные настройки в ~/.zshrc
echo 'export LANG=ru_RU.UTF-8' >> ~/.zshrc
echo 'export LC_ALL=ru_RU.UTF-8' >> ~/.zshrc
```

#### **9.2 Проблемы с Windows Terminal**  
Если появляется ошибка "При загрузке параметров пользователя возникли ошибки" с упоминанием недопустимого пути к иконке:

1. Откройте настройки Windows Terminal (Ctrl+,)
2. Найдите и отредактируйте JSON-файл
3. Найдите упоминания "icon" и укажите правильный путь или удалите эту строку

```json
// Примерный фрагмент с ошибкой
{
  "profiles": {
    "list": [
      {
        "name": "PowerShell",
        "commandline": "pwsh.exe",
        "icon": "исправьте/или/удалите/этот/путь.png"
      }
    ]
  }
}
```

---

### **❗ Важные примечания**  
1. **Драйверы NVIDIA**:  
   - Устанавливаются **только в Windows** (версия ≥570.00 для RTX 4090).  
   - В Debian пакеты CUDA и NVIDIA нужны только для утилит и библиотек.  

2. **Группы пользователя**:  
   - Группы `audio`, `video` **не требуются** — доступ к оборудованию управляется через Windows.  

3. **WSLg**:  
   - Для GUI-приложений убедитесь, что в Windows установлен [драйвер WSLg](https://learn.microsoft.com/en-us/windows/wsl/wslg).  

4. **Пакеты для разработки**:  
   - Для установки компиляторов и средств разработки используйте `build-essential` вместо отдельных пакетов.

---

### **🔗 Актуальные ссылки**  
- [Документация Microsoft по WSL](https://learn.microsoft.com/en-us/windows/wsl/)  
- [NVIDIA CUDA в WSL](https://docs.nvidia.com/cuda/wsl-user-guide/index.html)  
- [Документация Docker с NVIDIA](https://docs.docker.com/config/containers/resource_constraints/#gpu)  

**Для обновления системы (2025):**  
```bash
sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y
```
