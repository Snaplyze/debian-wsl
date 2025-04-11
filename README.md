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

#### **2.2 Вариант 2: Ручная установка с выбором расположения**  
Поскольку структура установочных файлов может меняться, наиболее надежный способ - использовать стандартный механизм установки с последующим перемещением:

```powershell
# Шаг 1: Установка Debian из Microsoft Store без запуска
wsl --install debian --no-launch

# Шаг 2: Создание папки для хранения дистрибутива
mkdir -p C:\WSL\Debian

# Шаг 3: Экспорт установленного образа в tar-файл
wsl --export Debian C:\WSL\debian-backup.tar

# Шаг 4: Удаление исходного дистрибутива
wsl --unregister Debian

# Шаг 5: Импорт в выбранное местоположение
wsl --import Debian C:\WSL\Debian C:\WSL\debian-backup.tar --version 2

# Шаг 6: Удаление временного tar-файла
Remove-Item C:\WSL\debian-backup.tar
```

Этот метод гарантированно работает независимо от изменений в структуре установочных файлов.

#### **2.3 Изменение местоположения уже установленного дистрибутива**  
Если вы уже установили Debian с помощью `wsl --install debian` и хотите изменить расположение:

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

# Настройка лимитов ресурсов (опционально)
echo "[wsl2]" > ~/wslconfig.txt
echo "memory=8GB" >> ~/wslconfig.txt
echo "processors=4" >> ~/wslconfig.txt
echo "swap=4GB" >> ~/wslconfig.txt
echo "" >> ~/wslconfig.txt
echo "# Скопируйте этот файл в %USERPROFILE%\.wslconfig на Windows" >> ~/wslconfig.txt
echo "# Пример: copy ~/wslconfig.txt /mnt/c/Users/username/.wslconfig" >> ~/wslconfig.txt
```

**Примечание**: 
- После изменения файла `/etc/wsl.conf` необходимо перезапустить WSL командой `wsl --shutdown` и запустить его снова.
- Секция `[interop]` управляет взаимодействием между Windows и Linux:
  - `enabled=true`: Позволяет запускать Windows-программы из Linux (например, `notepad.exe`)
  - `appendWindowsPath=true`: Добавляет пути Windows в переменную $PATH Linux, что позволяет вызывать Windows-программы без указания полного пути

---

### **4. Создание пользователя**  
```bash
# Установка fish shell
sudo apt install -y fish sudo

# Установка fish shell по умолчанию для root
sudo chsh -s /usr/bin/fish root

# Создание пользователя wsluser
sudo useradd -m -G sudo -s /usr/bin/fish wsluser

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
sudo apt install -y nvidia-container-toolkit

# Установка CUDA инструментов (имена пакетов могут отличаться в зависимости от версии Debian)
sudo apt install -y nvidia-cuda-toolkit || sudo apt install -y cuda

# Если вам нужна конкретная версия CUDA (опционально)
sudo apt search cuda | grep cuda-toolkit
# Выберите подходящую версию, например:
# sudo apt install -y cuda-toolkit-12-3
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
    python3-venv \
    htop \
    curl \
    wget \
    unzip \
    git \
    build-essential \
    pkg-config
```

#### **6.2 Fish Shell с популярными плагинами (для версии 4.0.0+)**  

**Популярные плагины и утилиты для Fish:**
1. **Fisher** - менеджер плагинов для Fish
2. **z** - умная навигация по часто используемым директориям
3. **fzf.fish** - интеграция нечеткого поиска файлов и команд
4. **done** - уведомления о завершении длительных команд
5. **bass** - запуск bash-скриптов и функций в Fish
6. **autopair** - автоматическое закрытие скобок и кавычек

```bash
# Установка дополнительных инструментов
sudo apt install -y fzf fd-find bat

# Создание директорий для конфигурации
mkdir -p ~/.config/fish/functions
mkdir -p ~/.config/fish/completions

# Настройка fish для пользователя - добавляем в конец существующего файла
echo '# Настройки WSL Debian' >> ~/.config/fish/config.fish
echo 'set -gx LANG ru_RU.UTF-8' >> ~/.config/fish/config.fish
echo 'set -gx LC_ALL ru_RU.UTF-8' >> ~/.config/fish/config.fish
echo '' >> ~/.config/fish/config.fish
echo '# Алиасы' >> ~/.config/fish/config.fish
echo "alias ll='ls -la'" >> ~/.config/fish/config.fish
echo "alias la='ls -A'" >> ~/.config/fish/config.fish
echo "alias l='ls'" >> ~/.config/fish/config.fish
echo "alias cls='clear'" >> ~/.config/fish/config.fish
echo "alias ..='cd ..'" >> ~/.config/fish/config.fish
echo "alias ...='cd ../..'" >> ~/.config/fish/config.fish
echo '' >> ~/.config/fish/config.fish
echo '# Улучшенные утилиты' >> ~/.config/fish/config.fish
echo "type -q batcat && alias cat='batcat --paging=never'" >> ~/.config/fish/config.fish
echo "type -q fd && alias find='fd'" >> ~/.config/fish/config.fish
echo '' >> ~/.config/fish/config.fish
echo '# Настройка fish' >> ~/.config/fish/config.fish
echo 'set -U fish_greeting' >> ~/.config/fish/config.fish
echo 'set fish_key_bindings fish_default_key_bindings' >> ~/.config/fish/config.fish
echo 'set fish_autosuggestion_enabled 1' >> ~/.config/fish/config.fish
echo '' >> ~/.config/fish/config.fish
echo '# FZF интеграция' >> ~/.config/fish/config.fish
echo "set -gx FZF_DEFAULT_COMMAND 'fd --type f --strip-cwd-prefix 2>/dev/null || find . -type f'" >> ~/.config/fish/config.fish
echo 'set -gx FZF_CTRL_T_COMMAND $FZF_DEFAULT_COMMAND' >> ~/.config/fish/config.fish

# Создание приветственного сообщения
echo 'function fish_greeting' > ~/.config/fish/functions/fish_greeting.fish
echo '    echo "🐧 WSL Debian - $(date '\''+%Y-%m-%d %H:%M'\'')"' >> ~/.config/fish/functions/fish_greeting.fish
echo 'end' >> ~/.config/fish/functions/fish_greeting.fish

# Настройка завершений для Docker
curl -sL https://raw.githubusercontent.com/docker/cli/master/contrib/completion/fish/docker.fish -o ~/.config/fish/completions/docker.fish
curl -sL https://raw.githubusercontent.com/docker/compose/master/contrib/completion/fish/docker-compose.fish -o ~/.config/fish/completions/docker-compose.fish

# Установка Fisher и плагинов
fish -c "curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher"
fish -c "fisher install jethrokuan/z"
fish -c "fisher install PatrickF1/fzf.fish"
fish -c "fisher install jorgebucaran/autopair.fish"
fish -c "fisher install franciscolourenco/done"
fish -c "fisher install edc/bass"

# Установка Starship и добавление в конфигурацию
curl -sS https://starship.rs/install.sh | sh -s -- -y
echo 'starship init fish | source' >> ~/.config/fish/config.fish

# Настройка Fish для root
sudo mkdir -p /root/.config/fish
sudo mkdir -p /root/.config/fish/functions

# Основные настройки для root
sudo bash -c 'echo "# Настройки WSL Debian" > /root/.config/fish/config.fish'
sudo bash -c 'echo "set -gx LANG ru_RU.UTF-8" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "set -gx LC_ALL ru_RU.UTF-8" >> /root/.config/fish/config.fish'
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
sudo bash -c 'echo "type -q fd && alias find='\''fd'\''" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "# Настройка fish" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "set -U fish_greeting" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "set fish_key_bindings fish_default_key_bindings" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "set fish_autosuggestion_enabled 1" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "# FZF интеграция" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "set -gx FZF_DEFAULT_COMMAND '\''fd --type f --strip-cwd-prefix 2>/dev/null || find . -type f'\''" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "set -gx FZF_CTRL_T_COMMAND \$FZF_DEFAULT_COMMAND" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "" >> /root/.config/fish/config.fish'
sudo bash -c 'echo "starship init fish | source" >> /root/.config/fish/config.fish'

# Приветствие для root
sudo bash -c 'echo "function fish_greeting" > /root/.config/fish/functions/fish_greeting.fish'
sudo bash -c 'echo "    echo \"🐧 WSL Debian [ROOT] - \$(date '\''+%Y-%m-%d %H:%M'\'')\""  >> /root/.config/fish/functions/fish_greeting.fish'
sudo bash -c 'echo "end" >> /root/.config/fish/functions/fish_greeting.fish'

# Копирование автозавершений Docker для root (опционально)
sudo mkdir -p /root/.config/fish/completions
sudo cp ~/.config/fish/completions/docker.fish /root/.config/fish/completions/ 2>/dev/null || true
sudo cp ~/.config/fish/completions/docker-compose.fish /root/.config/fish/completions/ 2>/dev/null || true
```

**Примечание:**
- Команды можно копировать блоками для удобного выполнения через терминал
- Конфигурация добавляется в конец существующих файлов, не перезаписывая их
- Для установки плагинов используется команда `fish -c`, чтобы запустить fish в отдельной сессии

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

# Создание директории для конфигурации Docker
sudo mkdir -p /etc/docker

# Создание конфигурационного файла для NVIDIA GPU
sudo bash -c 'echo "{" > /etc/docker/daemon.json'
sudo bash -c 'echo "  \"runtimes\": {" >> /etc/docker/daemon.json'
sudo bash -c 'echo "    \"nvidia\": {" >> /etc/docker/daemon.json'
sudo bash -c 'echo "      \"path\": \"nvidia-container-runtime\"," >> /etc/docker/daemon.json'
sudo bash -c 'echo "      \"runtimeArgs\": []" >> /etc/docker/daemon.json'
sudo bash -c 'echo "    }" >> /etc/docker/daemon.json'
sudo bash -c 'echo "  }," >> /etc/docker/daemon.json'
sudo bash -c 'echo "  \"default-runtime\": \"nvidia\"" >> /etc/docker/daemon.json'
sudo bash -c 'echo "}" >> /etc/docker/daemon.json'

# Проверка созданного файла
sudo cat /etc/docker/daemon.json

# Добавление пользователя в группу docker
sudo usermod -aG docker wsluser

# Настройка systemd для Docker
sudo systemctl enable docker
sudo systemctl start docker
```

**Примечание**: После добавления пользователя в группу docker, нужно перезапустить сессию или выполнить `su - wsluser`, чтобы изменения вступили в силу.

---

### **7. Проверка системы**  
#### **7.1 NVIDIA и CUDA**  
```bash
# Проверка доступности NVIDIA GPU
nvidia-smi

# Проверка CUDA
nvcc --version

# Проверка Docker с NVIDIA GPU (адаптируйте версию, если необходимо)
sudo docker run --rm --gpus all nvidia/cuda:latest-base nvidia-smi
```

#### **7.2 Локализация и время**  
```bash
# Проверка настроек локализации
locale

# Проверка часового пояса
timedatectl | grep "Time zone"
```

#### **7.3 Fish и Docker**  
```bash
# Перезапуск WSL для применения всех настроек
# В PowerShell на Windows:
# wsl --terminate Debian
# wsl -d Debian

# Проверка текущей оболочки
echo $SHELL

# Проверка Docker
docker ps

# Проверка автозавершения Fish
fish -c "type docker"
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
# Исправление проблем с правами доступа
mkdir -p ~/.config/fish/functions
chmod -R 755 ~/.config/fish/functions
chmod -R 755 ~/.config/fish/completions

# Обновление индекса завершений
fish -c "fish_update_completions"

# Если powerline-шрифты отображаются некорректно
mkdir -p ~/.local/share/fonts
curl -fLo ~/.local/share/fonts/MesloLGS-NF-Regular.ttf https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf
```

#### **9.1 Проблемы с русской локализацией**
Если русский язык не работает после всех настроек:

```bash
# Проверка доступных локалей
locale -a | grep ru

# Дополнительные настройки в fish
echo 'set -x LANG ru_RU.UTF-8' >> ~/.config/fish/config.fish
echo 'set -x LC_ALL ru_RU.UTF-8' >> ~/.config/fish/config.fish
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

### **10. Удаление WSL-дистрибутива**

Если вам нужно полностью удалить дистрибутив Debian из WSL (например, для чистой установки):

```powershell
# В PowerShell с правами администратора

# Остановка работающего дистрибутива
wsl --terminate Debian

# Удаление дистрибутива
wsl --unregister Debian

# Проверка, что дистрибутив удален
wsl --list
```

**Дополнительная очистка** (если вы вручную устанавливали дистрибутив):
```powershell
# Удаление папки дистрибутива (путь может отличаться)
Remove-Item -Recurse -Force C:\WSL\Debian

# Очистка временных файлов (если вы скачивали .appx)
Remove-Item .\debian-wsl.appx
Remove-Item .\debian-wsl.zip
Remove-Item -Recurse -Force .\debian-wsl
```

После удаления вы можете заново установить дистрибутив, следуя инструкциям из раздела 2.

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
