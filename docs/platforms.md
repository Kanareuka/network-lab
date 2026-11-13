---
id: gitlab-netlab-platforms
type: project-guide
discipline: proektuvannia-avtonomykh-merezh
status: draft
version: 7
updated: '2026-11-13'
sources: []
---

# ПР3: встановлення середовища з нуля

Основний покроковий маршрут встановлення міститься в [P-01 ПР3](pr03.md#p-01-встановлення-й-власний-git-репозиторій). Цей довідник доповнює його альтернативними середовищами та доступом до панелей. Назви екранів інсталятора можуть змінюватися; посилання ведуть на офіційні інструкції з актуальними завантаженнями, перевірені 31.10.2026.

| Ваша ОС | Основний маршрут | Де виконувати Bash-команди ПР |
|---|---|---|
| Windows | Git for Windows + Docker Desktop із Linux Engine на WSL2 | Git Bash; з PowerShell — явний виклик Git Bash нижче |
| Windows, проєкт на диску Windows | Git for Windows + Docker Desktop із Linux Engine | Git Bash; з PowerShell — явний виклик Git Bash нижче |
| macOS | Docker Desktop для Apple Silicon або Intel відповідно до процесора | Terminal; для блоків ПР можна запустити bash |
| Linux | Docker Engine + Compose plugin у навчальній Ubuntu | Термінал цієї Ubuntu |
| Будь-яка ОС, обрана окрема VM | Ubuntu у сумісному гіпервізорі + Docker Engine | Термінал усередині VM |

**WSL2 і Hyper-V не треба встановлювати як два послідовні кроки.** WSL2 дає Linux-середовище у Windows, а Docker Desktop може використовувати його для Linux-контейнерів. Окрема Ubuntu VM у Hyper-V — альтернативний навчальний комп’ютер, усередині якого встановлюють Engine. Hyper-V є не в усіх редакціях Windows. Можна використати інший гіпервізор; важливі робочі Linux-контейнери, а не його назва. Не запускайте VM усередині WSL і другий Docker daemon усередині toolbox.

Слабкий ноутбук: Docker Desktop Windows документує 8 GB RAM як системну вимогу, Mac — 4 GB. Це вимоги продукту, а не доведений мінімум нашого стенда з LLM. Engine в окремій Linux/WSL-системі — альтернативний маршрут, який потрібно випробувати на конкретному ПК. Не виділяйте VM всю RAM хоста. Якщо обладнання не відповідає вимогам, погодьте навчальний Linux-комп’ютер із викладачем; не підміняйте локальний inference готовим текстом.

## Windows: Git Bash і Docker Desktop

Цей маршрут підходить, коли репозиторій зберігається на диску Windows і ви працюєте з Docker Desktop. Git Bash запускає команди протоколів, а контейнери виконує Linux Engine Docker Desktop. Це окремий маршрут від команд усередині користувацької Ubuntu WSL.

1. Установіть [Git for Windows](https://gitforwindows.org/) та [Docker Desktop](https://docs.docker.com/desktop/setup/install/windows-install/). Після встановлення відкрийте новий термінал, щоб він отримав оновлений PATH.
2. Запустіть Docker Desktop і дочекайтеся готовності Engine. Потрібні **Linux containers**. Якщо меню Docker пропонує Switch to Linux containers, переключіть режим.
3. Створіть власну копію шаблону й клонуйте її за [git.md](git.md). У Провіднику відкрийте її корінь, оберіть Open Git Bash here. У Windows 11 за потреби спочатку натисніть «Показати додаткові параметри». Інший спосіб — відкрити Git Bash із меню Start і перейти до каталогу проєкту.
4. У Git Bash виконайте `pwd`, `ls`, `git --version`, `docker version`, `docker compose version` та `docker info --format '{{.OSType}}'`. `ls` має показати compose.yaml і tests; Docker — Client і Server, тип Engine — linux. Якщо Server недоступний, спочатку виправте підключення до Docker Desktop.
5. Заповніть Dockerfile і compose.yaml за P-02 ПР3, потім виконайте блок запуску й тестів із [P-02 ПР3](pr03.md#p-02-створити-dockerfile-і-compose). Порожні TODO не можуть дати 10/10.

Усі блоки `bash` у протоколах виконуйте в цьому Git Bash. Якщо в PowerShell є команда `bash` або `wsl`, вона може запустити інше Linux-середовище. Для PowerShell використайте явний шлях до Git Bash у наступному розділі.

### PowerShell: запуск того самого стенда

Ці команди виконуйте **в PowerShell, відкритому в корені власного репозиторію**, після заповнення Dockerfile і compose.yaml у ПР3. Стандартний шлях Git Bash визначаємо через ProgramFiles:

```powershell
$netlabBash = Join-Path $env:ProgramFiles 'Git/bin/bash.exe'
Test-Path $netlabBash
docker version
docker compose version
docker info --format '{{.OSType}}'
```

Очікується `True`, Client і Server та `linux`. Якщо `False`, Git установлено в інший каталог або ще не встановлено: відкрийте Git Bash з меню Start та скористайтеся Bash-командами ПР3. Якщо Docker показує помилку, запустіть Desktop і виправте підключення до Engine перед продовженням. Права адміністратора для звичайного запуску стенда не потрібні, якщо доступ до Docker вже налаштовано.

Виконуйте наступні команди по одній. Після кожного виклику перевіряйте повідомлення; при помилці виправте її, потім повторіть цей крок:

```powershell
if (-not (Test-Path '.env')) { Copy-Item '.env.example' '.env' }
& $netlabBash -c 'sh tests/compose.sh pr03 config'
& $netlabBash -c 'sh tests/compose.sh pr03 up -d --build'
& $netlabBash -c 'sh tests/compose.sh pr03 ps'
& $netlabBash -c 'sh tests/check.sh pr03'
```

| Команда | Результат і наступна дія |
|---|---|
| Умова з Copy-Item | Створює .env лише за його відсутності; уже налаштований файл зберігається |
| config | Друкує валідну конфігурацію; помилка YAML потребує виправлення файлів до up |
| up -d --build | Будує ваш toolbox і запускає його у фоновому режимі |
| ps | Toolbox має стан Up/Running; при Exited прочитайте logs |
| check.sh pr03 | Виконує 7 перевірок; повний успіх дає 10/10 і створює docs/reports/pr03.md |

Код завершення останнього виклику Git Bash можна одразу подивитися через `$LASTEXITCODE`: `0` — успіх, інше значення — помилка. При несправних передумовах автопротокол пояснює проблему й не визначає остаточний бал. Порожній журнал toolbox із командою sleep є нормальним; це ще не доказ успіху тестів.

Діагностика, зупинка й повторний запуск у тому самому PowerShell:

```powershell
& $netlabBash -c 'sh tests/compose.sh pr03 logs --tail 30 toolbox'
& $netlabBash -c 'sh tests/compose.sh pr03 stop'
& $netlabBash -c 'sh tests/compose.sh pr03 up -d'
& $netlabBash -c 'sh tests/check.sh pr03'
```

Після нового відкриття PowerShell повторно задайте `$netlabBash`. `.env` та конфігурації повторно створювати не потрібно. У Git додайте згенерований docs/reports/pr03.md разом із власними конфігураціями й надішліть гілку/тег; для здачі потрібен тільки URL репозиторію.

Якщо IMAGES повідомляє про невідповідний digest, номерний тег міг отримати нову збірку або локально залишився попередній образ. За актуального узгодженого lock-файла оновіть кеш образів ПР3 й повторіть побудову та тест:

```powershell
docker pull python:3.15.2-alpine
docker pull docker:29.9.2-cli
& $netlabBash -c 'sh tests/compose.sh pr03 up -d --build'
& $netlabBash -c 'sh tests/check.sh pr03'
```

Це теги поточної редакції з .env.example. Якщо після pull перевірка знову не проходить, передайте викладачу текст IMAGES і отримайте перевірене оновлення шаблону. Самостійно підміняти очікуваний digest або вимикати перевірку не потрібно.

## Windows: Ubuntu у WSL2

1. Відкрийте PowerShell із правами адміністратора **для встановлення WSL**. Виконайте:

```powershell
wsl --install -d Ubuntu
```

Команда встановлює WSL та Ubuntu, а не сам Docker. Дочекайтеся завершення; якщо Windows попросить перезапуск, виконайте його. Відкрийте Ubuntu з меню Start, створіть Linux-ім’я користувача та пароль. Під час введення Linux-пароля символи не показуються — це нормально. Далі у PowerShell перевірте:

```powershell
wsl --update
wsl --list --verbose
```

Очікується рядок Ubuntu з VERSION 2. Якщо VERSION 1, виконайте `wsl --set-version Ubuntu 2`; ім’я дистрибутива візьміть точно з виводу. Якщо повідомляється про вимкнену віртуалізацію, виконайте рекомендацію Microsoft для вашого ПК/UEFI; Dockerfile цю проблему не виправить. [Офіційна інструкція WSL](https://learn.microsoft.com/en-us/windows/wsl/install).

2. Відкрийте [Docker Desktop Windows](https://docs.docker.com/desktop/setup/install/windows-install/), звірте вимоги своєї ОС і завантажте відповідний інсталятор. Для звичайного Intel/AMD ПК потрібний x86_64; Windows ARM має окремий статус підтримки в офіційній документації. Під час встановлення використайте WSL2 backend, якщо інсталятор пропонує вибір. Запустіть Docker Desktop і дочекайтеся готовності engine.

3. У Settings → Resources → WSL Integration увімкніть інтеграцію з вашою Ubuntu, застосуйте зміни. Якщо розділ недоступний, перевірте, що Desktop працює з Linux containers. Повний Hyper-V як окрема VM для цього маршруту не потрібний. Не встановлюйте паралельно Engine всередині тієї ж Ubuntu, коли користуєтеся Desktop integration: два daemon ускладнять розуміння, де ваші контейнери.

4. Перевірте Docker у вибраній Ubuntu командою з наступного блока. Git для Windows-маршруту встановлюється у Windows за P-01 ПР3; clone, commit і push виконуйте через Git Bash. Наявну копію проєкту у Windows не потрібно повторно клонувати в Ubuntu для запуску скриптів.

Якщо викликаєте Ubuntu з PowerShell, спочатку перевірте `wsl --list --verbose`: зірочка позначає дистрибутив за замовчуванням. Службовий `docker-desktop` обслуговує Docker Desktop і не є навчальною Ubuntu. Сам факт його наявності не означає готовність Ubuntu-маршруту. Перевірте Docker у потрібному дистрибутиві явно:

```powershell
wsl -d Ubuntu -- docker version
```

Тут `Ubuntu` — приклад точної назви з вашого списку; якщо у вас `Ubuntu-24.04`, замініть значення на `Ubuntu-24.04`. Очікуються Client і Server. Інакше перевірте WSL Integration саме для цього дистрибутива. Далі відкрийте термінал цієї Ubuntu, перейдіть у свій репозиторій і виконуйте Bash-команди ПР3. Не використовуйте узагальнене `wsl sh tests/check.sh pr03`, доки не перевірили, який саме дистрибутив воно запустить. [Docker: інтеграція з WSL](https://docs.docker.com/desktop/features/wsl/), [Microsoft: вибір дистрибутива](https://learn.microsoft.com/en-us/windows/wsl/basic-commands).

## macOS: Docker Desktop і Git

У меню Apple → About This Mac перевірте Chip/Processor. Відкрийте [Docker Desktop macOS](https://docs.docker.com/desktop/setup/install/mac-install/). Оберіть Apple Silicon для M-серії або Intel для Intel Mac. Відкрийте .dmg, перемістіть Docker до Applications, запустіть і виконайте початкові підказки. Дочекайтеся готового engine. Не завантажуйте Linux .deb на macOS.

Відкрийте Terminal. Виконайте `git --version`. Якщо macOS пропонує встановити Command Line Tools, завершіть це встановлення або виконайте `xcode-select --install`, потім повторіть перевірку. Очікується git version ... . Для блоків ПР виконайте `bash` і працюйте в тому самому терміналі. На Apple Silicon використовуйте відповідний інсталятор Docker Desktop та перевірте готовність командами нижче. Якщо Docker повідомляє про несумісну архітектуру образу, передайте викладачу точний текст помилки.

## Ubuntu Linux або окрема VM: Docker Engine

Цей розділ виконується **всередині Ubuntu**, не в PowerShell і не в macOS Terminal хоста. Для нового навчального середовища використайте підтримувану Ubuntu LTS; станом на дату перевірки офіційна сторінка підтримує 24.04 і 26.04 LTS. Нижче маршрут через офіційний apt-репозиторій Docker. Якщо в системі вже є Docker/containerd від іншого постачальника, спершу прочитайте розділ Uninstall old versions офіційної сторінки; не видаляйте чужу робочу інсталяцію автоматично.

```bash
sudo apt update
sudo apt install -y ca-certificates curl git
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Ці команди встановлюють інструменти завантаження, створюють каталог ключів і додають публічний ключ репозиторію Docker для перевірки пакетів. /etc тут — стандартні системні файли **навчальної Ubuntu**, їх не треба створювати у Git-проєкті. Перевірте, що curl завершився без помилки, і додайте опис джерела пакетів:

```bash
. /etc/os-release
printf 'Types: deb\nURIs: https://download.docker.com/linux/ubuntu\nSuites: %s\nComponents: stable\nArchitectures: %s\nSigned-By: /etc/apt/keyrings/docker.asc\n' "${UBUNTU_CODENAME:-$VERSION_CODENAME}" "$(dpkg --print-architecture)" | sudo tee /etc/apt/sources.list.d/docker.sources
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
```

Перший рядок читає ідентифікатор вашої Ubuntu. printf і tee записують адресу stable-каналу пакунків Docker, кодову назву ОС і архітектуру; це **apt stable**, воно не пов’язане з Docker image тегом stable. Останні команди встановлюють engine, CLI, buildx і Compose plugin та запускають службу. Очікується активна служба Docker; перевірте `sudo systemctl status docker --no-pager`.

Для роботи без sudo перед кожною Docker-командою у власній навчальній Ubuntu:

```bash
sudo usermod -aG docker "$USER"
```

Вийдіть із Linux-сеансу й увійдіть знову; у WSL закрийте та повторно відкрийте термінал. Потім перевірте docker version без sudo. Група docker дає широкий доступ до engine; це налаштування власного навчального середовища. Не застосовуйте chmod 666 до docker.sock. [Офіційне встановлення Engine Ubuntu](https://docs.docker.com/engine/install/ubuntu/), [після встановлення](https://docs.docker.com/engine/install/linux-postinstall/).

Для Windows із дуже обмеженими ресурсами Engine у WSL2 без Desktop можна розглядати як окремий альтернативний маршрут: спочатку робоча Ubuntu WSL2 із systemd, потім розділ Engine вище. Не комбінуйте його з Desktop integration. Після налаштування перевірте готовність командами наступного розділу.

## Перевірка готовності для всіх маршрутів

```bash
git --version
docker version
docker compose version
docker info --format '{{.OSType}}'
```

| Команда | Очікується | Якщо інакше |
|---|---|---|
| git --version | git version ... | Git не встановлений або термінал відкритий до інсталяції |
| docker version | Обидва розділи Client і Server | Лише Client: запустіть engine; permission denied: перевірте групу/повторний вхід; Desktop: WSL integration |
| docker compose version | Docker Compose version ...; підтримується сучасний plugin | compose not found: установіть plugin або оновіть Desktop; старий docker-compose не підставляйте |
| docker info --format ... | linux | windows: переключіть Desktop на Linux containers; daemon error: спочатку виправте server |

У командах лабораторних не використовуйте sudo, якщо обрали налаштований користувацький доступ до Docker. Не створюйте весь репозиторій через sudo: файли стануть власністю root. Інсталяція, перезапуски й усунення помилок входять у активний час ПР3; скачування пакетів/образів обліковуйте окремо. Якщо встановлення забрало бюджет роботи, зафіксуйте це для викладача — не приховуйте зайві години.

## Окрема VM та Hyper-V

Якщо ви обрали цей маршрут, спершу створіть Ubuntu VM за документацією обраного гіпервізора, встановіть ОС і перевірте доступ до інтернету. Після цього виконуйте розділ Engine. На Windows [Hyper-V](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/get-started/Install-Hyper-V) — лише один можливий гіпервізор; він не є обов’язковою програмою курсу. Підтримка вкладеної віртуалізації не потрібна, коли Engine запускає контейнери безпосередньо в Ubuntu VM.

## Браузер і окрема VM

Порти стенда прив’язані до loopback. 127.0.0.1 усередині довільної VM означає цю VM, а не основну ОС. Використайте браузер усередині VM або SSH-тунель; відкривати панелі на всіх інтерфейсах не потрібно.

У власній Ubuntu VM за потреби встановіть SSH:

```bash
sudo apt update
sudo apt install openssh-server
systemctl status ssh
hostname -I
```

`hostname -I` показує адреси VM. Виберіть адресу інтерфейсу, доступного з основної ОС, за налаштуваннями мережі свого гіпервізора. `student` нижче — ваш користувач Ubuntu, а `VM_ADDRESS` — ця адреса; обидва позначення замініть. З основної ОС спочатку перевірте вхід командою `ssh student@VM_ADDRESS`; перше підтвердження ключа можливе після звіряння адреси. Вийдіть командою `exit`. Якщо NAT не дає прямого доступу, налаштуйте перенаправлення лише SSH (наприклад, host loopback 2222 → VM 22). Тепер у терміналі основної ОС запустіть тунель (`-N` не відкриває командну оболонку, `-L` перенаправляє локальний порт до порту VM):

```bash
ssh -N -o ExitOnForwardFailure=yes -L 127.0.0.1:8088:127.0.0.1:8088 -L 127.0.0.1:9090:127.0.0.1:9090 -L 127.0.0.1:3000:127.0.0.1:3000 student@VM_ADDRESS
```

Для NAT-перенаправлення на 2222 замість останнього аргументу використайте `-p 2222 student@127.0.0.1`. Тримайте термінал відкритим. Тепер браузер основної ОС бачить gateway на 8088, Prometheus на 9090 і Grafana на 3000, коли відповідний етап запущено. При зміні портів у .env змініть праві порти тунелю, при зайнятому host-порті — лівий порт і адресу браузера. LLM API публікувати не потрібно.

WSL окремо документує доступ із Windows через localhost; це не автоматичне правило для будь-якої VM. Після створення тунелю перевірте доступ до потрібної панелі браузером основної ОС.

Джерела: [OpenSSH -L/-N](https://man.openbsd.org/ssh), [Ubuntu OpenSSH](https://ubuntu.com/server/docs/how-to/security/openssh-server/), [Microsoft WSL networking](https://learn.microsoft.com/en-us/windows/wsl/networking).
