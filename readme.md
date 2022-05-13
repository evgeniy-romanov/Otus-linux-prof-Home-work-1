# Домашнее задание
Обновить ядро в базовой системе
В материалах к занятию есть методичка, в которой описана процедура обновления ядра из репозитория.
По данной методичке требуется выполнить необходимые действия.
Полученный в ходе выполнения ДЗ Vagrantfile должен быть залит в ваш git-репозиторий.
Для проверки ДЗ необходимо прислать ссылку на него.
Для выполнения ДЗ со * и ** вам потребуется сборка ядра и модулей из исходников.

## Введение
Выполнение действий приведенных в методичке позволит познакомиться с такими инструментами, как Vagrant и Packer, получить базовые навыки работы с системами контроля версий (Github). Получить навыки создания кастомных образов виртуальных машин и основам их распространения через репозиторий Vagrant Cloud. Так же вы получите навыки по обновлению ядра системы из репозитория.

Все ниже описанные действия производятся на компьютере с установленным Debian Jessie и будут воспроизводимы на Ubuntu или Mint. Если у вас установлена другая ОС, то необходимо самостоятельно внести изменения на некоторых этапах.

Для выполнения работы потребуются следующие инструменты:

VirtualBox - среда виртуализации, позволяет создавать и выполнять виртуальные машины;
Vagrant - ПО для создания и конфигурирования виртуальной среды. В данном случае в качестве среды виртуализации используется VirtualBox;
Packer - ПО для создания образов виртуальных машин;
Git - система контроля версий
А так же аккаунты:

### GitHub https://github.com/

### Vagrant Cloud https://app.vagrantup.com

### Установка ПО

##### Vagrant

Переходим на https://www.vagrantup.com/downloads.html выбираем соответствующую версию. В данном случае Debian 64-bit и версия 2.2.6. Копируем ссылку и в консоли выполняем:

curl -O https://releases.hashicorp.com/vagrant/2.2.6/vagrant_2.2.6_x86_64.deb && \
sudo dpkg -i vagrant_2.2.6_x86_64.deb
После успешного окончания будет установлен Vagrant.

##### Packer
Переходим на https://www.packer.io/downloads.html выбираем соответствующую версию. В данном случае Linux 64-bit и версия 1.4.4. Копируем ссылку и в консоли выполняем:

curl https://releases.hashicorp.com/packer/1.4.4/packer_1.4.4_linux_amd64.zip | \
sudo gzip -d > /usr/local/bin/packer && \
sudo chmod +x /usr/local/bin/packer
После успешного окончания будет установлен Packer.
## Kernel update
Клонирование и запуск
Для запуска рабочего виртуального окружения необходимо зайти через браузер в GitHub под своей учетной записью и выполнить fork данного репозитория: https://github.com/dmitry-lyutenko/manual_kernel_update

После этого данный репозиторий необходимо склонировать к себе на рабочую машину. Для этого воспользуемся ранее установленным приложением git, при этом в <user_name> будет имя уже вашего репозитрия:

mkdir romanov
cd romanov
git clone git@github.com:evgeniy-romanov/manual_kernel_update.git
В текущей директории появится папка с именем репозитория. В данном случае manual_kernel_update. Ознакомимся с содержимым:

cd manual_kernel_update
ls -1
manual
packer
Vagrantfile
Здесь:

manual - директория с данным руководством
packer - директория со скриптами для packer'а
Vagrantfile - файл описывающий виртуальную инфраструктуру для Vagrant
Запустим виртуальную машину и залогинимся:

vagrant up
...
==> kernel-update: Importing base box 'centos/7'...
...
==> kernel-update: Booting VM...
...
==> kernel-update: Setting hostname...

vagrant ssh
[vagrant@kernel-update ~]$ uname -r
3.10.0-957.12.2.el7.x86_64
Теперь приступим к обновлению ядра.

kernel update
Подключаем репозиторий, откуда возьмем необходимую версию ядра.

sudo yum install -y http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
В репозитории есть две версии ядер kernel-ml и kernel-lt. Первая является наиболее свежей стабильной версией, вторая это стабильная версия с длительной поддержкой, но менее свежая, чем первая. В данном случае ядро 5й версии будет в kernel-ml.

Поскольку мы ставим ядро из репозитория, то установка ядра похожа на установку любого другого пакета, но потребует явного включения репозитория при помощи ключа --enablerepo.

#### Ставим последнее ядро:

sudo yum --enablerepo elrepo-kernel install kernel-ml -y
grub update
После успешной установки нам необходимо сказать системе, что при загрузке нужно использовать новое ядро. В случае обновления ядра на рабочих серверах необходимо перезагрузиться с новым ядром, выбрав его при загрузке. И только при успешно прошедших загрузке нового ядра и тестах сервера переходить к загрузке с новым ядром по-умолчанию. В тестовой среде можно обойти данный этап и сразу назначить новое ядро по-умолчанию.

Обновляем конфигурацию загрузчика:

sudo grub2-mkconfig -o /boot/grub2/grub.cfg
Выбираем загрузку с новым ядром по-умолчанию:

sudo grub2-set-default 0
Перезагружаем виртуальную машину:

sudo reboot
После перезагрузки виртуальной машины (3-4 минуты, зависит от мощности хостовой машины) заходим в нее и выполняем:

uname -r
5.17.7-1.el7.elrepo.x86_64

## Packer
Теперь необходимо создать свой образ системы, с уже установленым ядром 5й версии. Для это воспользуемся ранее установленной утилитой packer. В директории packer есть все необходимые настройки и скрипты для создания необходимого образа системы.

packer provision config
Файл centos.json содержит описание того, как произвольный образ. Полное описание можно найти в документации к packer. Обратим внимание на основные секции или ключи.

Создаем переменные (variables) с версией и названием нашего проекта (artifact):

    "artifact_description": "CentOS 7.7 with kernel 5.x",
    "artifact_version": "7.7.1908",
В секции builders задаем исходный образ, для создания своего в виде ссылки и контрольной суммы. Параметры подключения к создаваемой виртуальной машине.

    "iso_url": "http://mirror.yandex.ru/centos/7.7.1908/isos/x86_64/CentOS-7-x86_64-Minimal-1908.iso",
    "iso_checksum": "9a2c47d97b9975452f7d582264e9fc16d108ed8252ac6816239a3b58cef5c53d",
    "iso_checksum_type": "sha256",
В секции post-processors указываем имя файла, куда будет сохранен образ, в случае успешной сборки

    "output": "centos-{{user `artifact_version`}}-kernel-5-x86_64-Minimal.box",
В секции provisioners указываем каким образом и какие действия необходимо произвести для настройки виртуальой машины. Именно в этой секции мы и обновим ядро системы, чтобы можно было получить образ с 5й версией ядра. Настройка системы выполняется несколькими скриптами, заданными в секции scripts.

    "scripts" : 
      [
        "scripts/stage-1-kernel-update.sh",
        "scripts/stage-2-clean.sh"
      ]
Скрипты будут выполнены в порядке указания. Первый скрипт включает себя набор команд, которые мы ранее выполняли вручную, чтобы обновить ядро. Второй скрипт занимается подготовкой системы к упаковке в образ. Она заключается в очистке директорий с логами, временными файлами, кешами. Это позволяет уменьшить результирующий образ. Более подробно можно ознакомиться с ними в директории packer/scripts

Секция post-processors описывает постобработку виртуальной машины при ее выгрузке. Мы указыаем имя файла, в который будет сохранен результат (artifact). Обратите внимание, что имя задается на основе ранее созданной пользовательской переменной artifact_version значение которой мы задали ранее:

    "output": "centos-{{user `artifact_version`}}-kernel-5-x86_64-Minimal.box",

#### Установка локального пакера на удаленной машине:

cd /home/student/romanov/manual_kernel_update/packer
curl -O https://releases.hashicorp.com/packer/1.8.0/packer_1.8.0_linux_amd64.zip
unzip packer_1.8.0_linux_amd64.zip
./packer -version
1.8.0

ls -la

-rw-rw-r--. 1 student student      2009 May 12 16:46 centos.json
drwxrwxr-x. 2 student student        24 May 12 16:46 http
-rwxr-xr-x. 1 student student 151683072 Mar  4 10:56 packer
-rw-rw-r--. 1 student student  32195429 May 12 16:52 packer_1.8.0_linux_amd64.zip
drwxrwxr-x. 2 student student        62 May 12 16:54 scripts
##### Необходимо выполнить команду fix, чтобы убедиться, что ваши шаблоны работают с новой версией:
./packer fix centos.json > centos1.json

Далее редактируем файл centos1.json

vim centos1.json

было
"vm_name": "packer-centos-vm",

стало
"vm_name": "packer-centos-vm",
"headless": true

### packer build

Для создания образа системы достаточно перейти в директорию packer и в ней выполнить команду:

./packer build centos.json

[student@pv-homeworks1-10 packer]$ pwd
/home/student/romanov/manual_kernel_update/packer

[student@pv-homeworks1-10 packer]$ ls -la
total 1002548
drwxrwxr-x. 4 student student       173 May 12 19:39 .
drwxrwxr-x. 6 student student        81 May 12 20:22 ..
-rwxrwxr-x. 1 student student      2074 May 12 16:57 centos1.json
-rw-rw-r--. 1 student student 842718613 May 12 17:26 centos-7.7.1908-kernel-5-x86_64-Minimal.box
-rw-rw-r--. 1 student student      2009 May 12 16:46 centos.json
drwxrwxr-x. 2 student student        24 May 12 16:46 http
-rwxr-xr-x. 1 student student 151683072 Mar  4 10:56 packer
-rw-rw-r--. 1 student student  32195429 May 12 16:52 packer_1.8.0_linux_amd64.zip
drwxrwxr-x. 2 student student        62 May 12 16:54 scripts

## Vagrant cloud

Поделимся полученным образом с сообществом. Для этого зальем его в Vagrant Cloud. Можно залить через web-интерфейс, но так же vagrant позволяет это проделать через CLI. Логинимся в vagrant cloud, указывая e-mail, пароль и описание выданого токена (можно оставить по-умолчанию)

vagrant cloud auth logout - отвязаться от учётной записи
vagrant cloud auth login - привязаться к учётной записи
Vagrant Cloud username or email: <user_email>
Password (will be hidden): 
Token description (Defaults to "Vagrant login from DS-WS"): - пароль сгенерированный под вашим аккаунтом Vagrant Cloud
You are now logged in.

Теперь публикуем полученный бокс:

[student@pv-homeworks1-10 packer]$ vagrant cloud publish --release evgeniy-romanov86/centos-7-5 1.0 virtualbox centos-7.7.1908-kernel-5-x86_64-Minimal.box

### Ссылка на созданный бокс 
https://app.vagrantup.com/evgeniy-romanov86/boxes/centos-7-5



