Запускаем виртуальную машину с помощью настроенного файла Vagrantfile. 

После успешного запуска логинимся в эту машину:

vagrant ssh

Приступаем к процедуре установки нового ядра из репозитория elrepo. Подключаем репозиторий и ставим ядро kernel-ml


sudo yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm

устанавливаем ядро, временно подключив установленный репозиторий:

sudo yum install --enablerepo=elrepo-kernel kernel-ml kernel-ml-devel devel пакет с исходниками ставим для последующих тестов, потом удалим



Обновляем конфигурацию загрузчика:

sudo grub2-mkconfig -o /boot/grub2/grub.cfg


Выбираем загрузку с новым ядром по-умолчанию:

sudo grub2-set-default 0




и сразу с помощью плагина sahara делаем снэпшот текущего состояния, на всякий случай, 


elfutils-libelf-devel
ncurses-devel
openssl-devel

