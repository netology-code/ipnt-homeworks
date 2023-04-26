## Инструкция по подготовке ВМ c Asterisk

В данной инструкции используется VirtualBox. Вы можете использовать любую среду виртуализации. 

### Подготовка

1. Скачайте *.ova файл виртуальной машины и запустите скаченный файл.

2. Соглашайтесь со всем, что предлагает Virtualbox. Далее в списке виртуальных машин появится Netology_ntw_10.04.

![image](https://user-images.githubusercontent.com/77622076/234555396-dd5928db-9e42-4792-93d8-200c21f5b589.png)

3. Перейдите в режим настройки виртуальной машины: «Настроить»->«Сеть»->«Сетевой мост»:

![image](https://user-images.githubusercontent.com/77622076/234555739-74079b7a-0bde-46ce-b259-9b66cf5cae96.png)

4. После выбора сетевой установки типа подключения «Сетевой мост» убедитесь, что в надстройке «имя» выбран интерфейс физической машины. На интерфейсе доступна сети «Интернет» и желательно ваш домашний роутер осуществляет выдачу DHCP. 

![image](https://user-images.githubusercontent.com/77622076/234558071-7cb1ac45-27a7-4f39-a09f-9d6b7cfb46f6.png)

5. Нажмите «ОК» и запустите виртуальную машину Asterisk_NTW_10-04. Вход осуществляем под пользователем root пароль 1234.

6. Убедитесь, что получили IPv4 адрес с домашнего роутера: для этого введите команду `ip a `

![image](https://user-images.githubusercontent.com/77622076/234559424-59cc6b85-74bd-4eb3-aad2-ed488fbd0c8f.png)

7. Нас интересует интерфейс №2 enp0s3, и его  IPv4 адрес. В случае примера 192.168.3.47.   
Если  в вашей сети отсутствует раздача по DHCP, переходите к этапу 1.

**Важно:**   
1. Наименование интерфейса enp0s3 может отличаться. После перезагрузки виртуальной машины IPv4 адрес может измениться, учитывайте это при дальнейшей настройке.
2. В данной виртуальной машине  может быть не установлен пакет sudo. Можете установить, задав команду `apt install sudo`.

### Этап 1

1. Для настройки статического адреса в Linux почти всегда используется файл лежащий в директории /etc/network/interfaces. Откройте файл для редактирования (так как вы зашли под root, то не используйте sudo) nano /etc/network/interfaces.   
Должно получиться, как на скрине ниже:

![image](https://user-images.githubusercontent.com/77622076/234563403-0b210376-f63d-4114-915b-ac2a608ae342.png)

2. После строки *#The primary network interfaces* удалите весь текст, но запомните название интерфейса enp0s3 (он должен быть одинаковым с интерфесом №2 вывода команды ` ip a `)  и записываем следующее  

```
iface eth0 inet static      
address 192.168.1.15      
netmask 255.255.255.0      
gateway 192.168.1.1        
broadcast 192.168.0.255        
dns-nameservers 8.8.8.8  
```

Должно получится, как на скрине ниже:

![image](https://user-images.githubusercontent.com/77622076/234614333-40097c1e-fef1-4836-b240-b62ed3f9e7d3.png)

Горячие клавиши:
ctrl+o – сохранить файл, 
ctrl+x – закрыть и возвращаемся в cli.

далее даем команду, на перезапуск сетевой службы
systemctl restart networking
командой ip a проверяем, что интерфейс №2 в состоянии UP, если DOWN даем команду ifup enp0s3
Проверяем доступность яндекса ping ya.ru , если пинги не пошли, смотрим и правим конфигурацию файла resolv.conf, который отвечает за преобразование имен в семействе Linux
nano /etc/resolv.conf 
Дописываем nameserver 8.8.8.8 .

![image](https://user-images.githubusercontent.com/77622076/234615587-c1968b44-0203-4d69-9f27-9ae910fb41a8.png)

Сохраняем, закрываем. Должно все работать
На этом подготовка виртуальной машины для установки asterisk закончена. 
Для более удобной работы подключимся к нашей ВМ по ssh 
Открываем putty, выбираем ssh, вписываем IPv4 адрес нашего второго интерфейса. В случае примера это 192.168.3.47

![image](https://user-images.githubusercontent.com/77622076/234615847-89ad58c3-c42e-465d-9968-09ce6801760c.png)

Подключаемся и принимаем сертификат, нажав в на кнопку «Accept»

![image](https://user-images.githubusercontent.com/77622076/234616265-7cd0ce3b-4858-427b-8d9a-b475fe2fc0d9.png)

Так как подключение по ssh под пользователем root изначально запрещено, входим на ВМ под учетной записью user пароль 1234
Далее необходимо войти в пользователя root, так как sudo не установлен 
su root
Вводим пароль root 1234
Возвращаемся в домашнюю директорию root (опционально, для удобства)
cd ~ 

![image](https://user-images.githubusercontent.com/77622076/234631945-51c1a0df-c16e-4b85-95dd-c3bf7df57e61.png)

Теперь мы можем перейти к Этапу 2 и приступить к скачиванию установочного пакета Asterisk

### Этап 2

Для скачивания архива установщика будем использовать утилиту wget , по каким то причинам ее может не быть в стандартной реализации deb поэтому можно дать команду на ее установку
apt install wget 
Приступаем к скачиванию
wget https://downloads.asterisk.org/pub/telephony/certified-asterisk/asterisk-certified-18.9-current.tar.gz
!ВАЖНО: Переодически версии дистрибутива меняются, если к вам возвращается ошибка 404  Not Found -> необходимо зайти в браузер по ссылке: https://downloads.asterisk.org/pub/telephony/certified-asterisk/
И в наименовании архива asterisk-certified-18.9-current.tar.gz изменить версию, на актуальную


После того, как архив скачался, он попал в директорию в которой мы на момент скачивания находились
Командой ls можно посмотеть, действительно ли лежит архив 
Выглядит это так

![image](https://user-images.githubusercontent.com/77622076/234632423-2cdb9c3c-4c1a-417b-852c-9990499800b6.png)

Теперь мы можем приступить к распаковке архива

tar -xvf asterisk-certified-18.9-current.tar.gz
Командой ls убеждаемся, что все распаковалось

![image](https://user-images.githubusercontent.com/77622076/234632550-e56d8f3c-5d2f-4676-bc72-ca38799848bf.png)

Переходим в директорию с исполняемыми скриптами Asterisk
cd ./asterisk-certified-18.9-cert4/contrib/scripts/
Опционально можем посмотреть, лежит ли там у нас интересующий нас файл
ls -la | grep install_prereq
Запускаем этап установки
./install_prereq  install
На этапе установки появится запрос на установку кода региона страны. Убеждаемся, что уже установлена цифра «7» нажимаем ок

![image](https://user-images.githubusercontent.com/77622076/234632688-51036c5a-d810-4d6c-8321-769652827a16.png)

Так выглядит завершение установки

![image](https://user-images.githubusercontent.com/77622076/234632876-74dc29f2-6799-4c45-b7c2-f8bad5d35731.png)

Теперь необходимо запустить процесс сборки конфигурации
Для этого, для удобства переходим в домашнюю директорию root
cd ~
И переходим туда, где лежит файл /configure для его запуска
cd asterisk-certified-18.9-cert4/
Выполняем запуск сценариев скрипта configure
./configure
Должно быть так

![image](https://user-images.githubusercontent.com/77622076/234633108-df932254-0221-45f4-acf8-25f59c981560.png)

Теперь необходимо выбрать опции которые мы будем устанавливать
Выполняем команду 
make menuselect
Попадаем в меню

![image](https://user-images.githubusercontent.com/77622076/234633236-2b6053bc-73d9-46a3-be03-0e6c25d8a991.png)

Перемещение осуществляется стрелками, установка галочки с помощью Enter 
Выбираем раздел Call detail recording – убеждаемся что стоит галочка у модуля cdr_adaptive_odbc и ставим галочку у модуля cdr_odbc

![image](https://user-images.githubusercontent.com/77622076/234633332-6ab340a8-e630-4d6c-b037-53afe6224f42.png)

Далее переходим в channel drivers, листаем в самый низ и выбираем chan_sip

![image](https://user-images.githubusercontent.com/77622076/234633475-b5a05cc6-f8e6-4911-bdb4-3606fced6f41.png)

Сохраняем и выходим

Выполняем окончательную сборку Asterisk 
make && make install && make samples && make config && make install-logrotate
Дожидаемся процесса окончания установки
Должно быть так

![image](https://user-images.githubusercontent.com/77622076/234633643-91c13f39-a114-47a0-838f-2b7270aa26f8.png)

Возвращаемся в домашнюю директорию root cd ~
Выполняем /usr/sbin/asterisk
Проверяем статус Asterisk
systemctl status asterisk

![image](https://user-images.githubusercontent.com/77622076/234633787-988a6f34-7563-4ed4-b51f-74e2ec5a4d7f.png)

Если он не активен, запускаем
systemctl start asterisk
и снова убеждаемся, что он запустился
systemctl status asterisk
Должно быть так

![image](https://user-images.githubusercontent.com/77622076/234633919-b2f76f36-6757-4b22-a356-f9d5ddc61a5c.png)

Этап подготовки завершен
Можно приступать к выполнению задания.
!ВАЖНО: Для подключения к asterisk следует использовать 
/usr/sbin/asterisk -rv 

### Описание использования текстового редактора nano

ctrl+x – закрыть текстовый редактор.
ctrl+o – сохранить изменения в файле.
ctrl+w – поиск по файлу
# - внутри текста логически закрывает применение настроек данной строки, используется для комментариев.
Полное описание: https://www.opennet.ru/man.shtml?topic=nano&category=1&russian=2
Пример How-to: https://losst.pro/tekstovyj-redaktor-nano-v-linux-dlya-novichkov

![image](https://user-images.githubusercontent.com/77622076/234634257-716a7cb7-41a9-4fc0-b00a-de03d4e3880c.png)





