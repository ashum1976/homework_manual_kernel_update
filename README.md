###                                                                Полуавтоматическая установка ядра 

Запускаем виртуальную машину с помощью настроенного файла Vagrantfile. 

После успешного запуска логинимся в эту машину:

vagrant ssh

Приступаем к процедуре установки нового ядра из репозитория elrepo. Подключаем репозиторий и ставим ядро kernel-ml


*           sudo yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm

устанавливаем ядро, временно подключив установленный репозиторий:

*           sudo yum install --enablerepo=elrepo-kernel kernel-ml 

Обновляем конфигурацию загрузчика:

*           sudo grub2-mkconfig -o /boot/grub2/grub.cfg

Выбираем загрузку с новым ядром по-умолчанию:

*           sudo grub2-set-default 0  <------- * данной настройкой мы говорим загрузчику использовать первое ядро для загрузки (первым идет последнее по версии ядро).
*           или
*           sudo grubby --set-default /boot/config-5.10.11  <---- прямо указываем какое ядро будет "default"

Перезагружаемся с новым ядром

На этом установка нового ядра в полуавтоматическом режиме завершена.


###                                                             Ручная конфигурация и установка нового ядра из исходных кодов 

Запускаем виртуальную машину с помощью настроенного файла Vagrantfile, и обновляем.
Сразу с помощью плагина sahara делаем снэпшот текущего состояния, на всякий случай
 
-             vagrant sandbox on


Качаем ядро, на хостовую машину, с сайта kernel.org, на момент 29.01.2021, версия ядра stable:	5.10.11 https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.10.11.tar.xz , и забрасываем на гостевую систему, в домашний каталог пользователя scp -P 2222 vagrant@127.0.0.1:~

-           В процессе сборки выяснилось, что gcc, установленный в дистрибутиве centos7 староват для сборки этой версии ядра, поэтому была установлена немного новее версия gcc-7.4.0.tar.gz (спасибо                                                                                                                   сокурснику Nikolay Malinin  подкинул ссылку для установки https://stackoverflow.com/questions/36327805/how-to-install-gcc-5-3-with-yum-on-centos-7-2), можно и новее поставить, но хватит и этой версии.                Качаем забрасываем на гостевую систему, в домашний каталог пользователя scp -P 2222 vagrant@127.0.0.1:~

              Собираем новый gcc, процесс не сложный, по ссылке есть описание (для сборки понадобится текущий gcc и gcc-c++):

-               yum install libmpc-devel mpfr-devel gmp-devel zlib-devel - пакеты для сборки
-               tar -xzj "путь где файл лежит" -C "куда распаковать"
-               cd "куда распаковали"/gcc-7.4.0 && mkdir build && cd ./build
-               ../configure --with-system-zlib --disable-multilib --enable-languages=c,c++
-               make 
-               make install

Устанавливается новый gcc /usr/local/bin, библиотеки в:

                /usr/local/lib/gcc/x86_64-pc-linux-gnu/7.4.0/plugin
                /usr/local/lib/gcc/x86_64-pc-linux-gnu/7.4.0
                /usr/local/lib64

Создаём файл в папке /etc/ld.so.conf.d/gcc.conf  и прописываем полученные пути для библиотек. Для root-a правим переменную $PATH, в файле .bash_profile 

              PATH="/usr/local/bin:/opt/bin:$PATH"

vagrant sandbox commit - зафиксировали изменения, чтоб в случае чего откатится на эту точку, и не ставить по новой gcc и т.д.

Удаляем старый gcc и gcc-c++ 

Перегружаемся, заходим vagrant ssh смотрим версию gcc --version (7.4.0 должна быть)


Для успешной настройки и компиляции ядра, понадобится ещё немного пакетов:

 -          openssl-devel
 -          bison
 -          flex
 -          perl
 -          elfutils-libelf-devel
 -          ncurses-devel
  

  
Распаковать исходники ядра в папку /usr/src/kernel.

Проделаем действия, "только инсталяции нового ядра ( загрузчик не нужно обновлять )",  из пункта " Полуавтоматическая установка ядра", чтоб скопировать готовый .config файл для конфигурации нового ядра :) 

*           cp /boot/config-`uname -r` /usr/src/kernel/linux-5.10.11/.config
или
*           cp /boot/config-5.10.11....   /usr/src/kernel/linux-5.10.11/.config     

ядро  kernel-ml 5.10.11 можно удалить 

Зайти в распакованную папку linux-5.10.11

make menuconfig - конфигурация ядра, через текстовое меню

для включения файловой системы vboxsf, используемой для монтирования каталогов с хостовой системы в гостевую, в VirtualBox, нужно включить в ядро пару параметров:

                            Symbol: VBOXGUEST [=y]                                                                                                                                                                      
                            │ Type  : tristate                                                                                                                                                                             
                            │ Defined at drivers/virt/vboxguest/Kconfig:2                                                                                                                                                  
                            │   Prompt: Virtual Box Guest integration support                                                                                                                                              
                            │   Depends on: VIRT_DRIVERS [=y] && X86 [=y] && PCI [=y] && INPUT [=y]                                                                                                                        
                            │   Location:                                                                                                                                                                                 
                            │     -> Device Drivers                                                                                                                                                                     
                            │ (2)   -> Virtualization drivers (VIRT_DRIVERS [=y])


                            Symbol: VBOXSF_FS [=y]                                                                                                                                                                       
                            │ Type  : tristate                                                                                                                                                                             
                            │ Defined at fs/vboxsf/Kconfig:1                                                                                                                                                               
                            │   Prompt: VirtualBox guest shared folder (vboxsf) support                                                                                                                                   
                            │   Depends on: MISC_FILESYSTEMS [=y] && X86 [=y] && VBOXGUEST [=y]                                                                                                                            
                            │   Location:                                                                                                                                                                                  
                            │     -> File systems                                                                                                                                                                          
                            │ (3)   -> Miscellaneous filesystems (MISC_FILESYSTEMS [=y])                                                                                                                                   
                            │ Selects: NLS [=y]
  
сохранили .config


                       
                           make olddefconfig  <-----  Запуск настройки параметров с использованием сохранённой конфигурации и применением параметров по умолчанию, если в старом конфиге таких не было
                           make                         
                           make htmldocs  <--------опционально, сборка документации ( можно и pdfdocs). 
                           make install          <------- кроме установки сового ядра в boot каталог, сгенерирует  initrd(kernel version).img 
                           make modules_install

Обновляем конфигурацию загрузчика:

*                       sudo grub2-mkconfig -o /boot/grub2/grub.cfg

Выбираем загрузку с новым ядром по-умолчанию:

*                       sudo grub2-set-default 0        <------- * данной настройкой мы говорим загрузчику использовать первое ядро для загрузки (первым идет последнее по версии ядро).
*                       или
*                       sudo grubby --set-default /boot/config-5.10.11    <---- прямо указываем какое ядро будет "default"
                           



Пересборка box'а

В любой момент текущий образ виртуальной машины vagrant может упаковать в box-контейнер.
На самом деле это обыкновенный tar.gz, в котором лежат Vagrantfile и необходимые системе виртуализации компоненты
(в случае VirtualBox — диск в формате vmdk, файл экспорта настроек виртуальной машины ovf;
После подготовки виртуальной машины (компиляция нового ядра из исходников) достаточно выполнить в каталоге :

vagrant package --output=<some-box-file-name.box>
vagrant box add --name <box-name> <some-box-file-name.box> 


Дл загрузки готового образ, созданного packer-ом в репозиторий VagrantCloud, необходимо предварительно создать репозиторий с таким именем, например:

        vagrant cloud publish  --release --force  ashum1976/centos-7-5 29.01.2021  virtualbox  centos-7.7.1908-kernel-5-x86_64-Minimal.box 

где:
 -  cloud publish - загрузить образ в облако;
 -  release - указывает на необходимость публикации образа после загрузки;
 -  ashum1976/centos-7-5 - это репозиторой который уже должен быть создан на VagrantCloud !!!!
 -  29.01.2021 - версия образа;
 -  virtualbox - провайдер;
 -  centos-7.7.1908-kernel-5-x86_64-Minimal.box - имя файла загружаемого образа;


elfutils-libelf-devel
ncurses-devel
openssl-devel

