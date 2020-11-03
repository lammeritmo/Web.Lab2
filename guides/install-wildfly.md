# Установка WildFly на helios

1. Подключитесь к терминалу на гелиосе
1. git clone https://github.com/filesRepository38/wildfly-for-helios
1. cd wildfly-for-helios/bin
1. Теперь можно добавить пользователя: выполните команду './add-user.sh' там в целом всё понятно, и, надеюсь, эта часть в пояснениях не нуждается
1. После этого можно запустить сервер приложений: './standalone.sh'

### или..

1. Скачайте wildfly и закиньте его при помощи scp или чего-нибудь ещё на гелиос
1. Подключиться к терминалу на гелиосе
1. Важная вещь, нужно поменять системную переменную JAVA_HOME, иначе wildfly будет ругаться на отсутствие каких-то опций, которые появились в jdk8, и не будет запускаться.  
Нужно открыть файл "~/.profile" для правки, т.е. команда vim ~/.profile. В этом файле помимо первой строчки . /usr/local/etc/profile  
добавить ещё две:  
export PATH=/usr/jdk/jdk1.8.0/bin:$PATH  
export JAVA_HOME=/usr/jdk/jdk1.8.0  
сохранить файл (для тех, кто не помнит wim - нажать Esc, нажать двоеточие (зажатым shift и клавишей Ж), набрать wq и нажать Enter);
затем выполнить команду  
. ~/.profile  
Она запустит изменения. В дальнейшем при входе на гелиос это будет делаться автоматически, этот пункт можно будет пропускать.
1. Теперь можно добавить пользователя: выполните команду './add-user.sh' там в целом всё понятно, и, надеюсь, эта часть в пояснениях не нуждается
1. После этого можно запустить сервер приложений: './standalone.sh'

# Настройка WildFly и деплой лаб
(Не забываем пробросить нужные нам порты(например, 8080 и 9990, если используете стандартную конфигурацию), чтобы можно было получить доступ к WildFly с локалки)

## Проброс портов

Так как на гелиосе все порты(или почти все) закрыты для доступа снаружи, необходимо выполнять проброс портов.  
На Linux, Mac OS и последних версиях 10-й винды доступна команда ssh, будем выполнять проброс через неё.

1. Заходим в терминал на вашем компьютере и вводим следующее
```shell
ssh -L [local_port]:localhost:[helios_port] s[isu_number]@helios.cs.ifmo.ru -p 2222
```
[local_port] - порт, который вы будете вводить у себя на ПК, для подлючения к чему-либо на гелиосе  
[helios_port] - порт на гелиосе, на котором что-либо будет запущено, например наш сервер приложений  
[isu_number] - ваш табельный номер  
Пример  
```shell
ssh -L 8080:localhost:8080 s265082@helios.cs.ifmo.ru -p 2222
```
1. Порт проброшен, можете пробовать подключаться

## Запуск в фоне
Вас также могут попросить запустить WildFly на фоне так, чтобы при закрытии терминала наш сервер приложений не выключился.  
Делается это при помощи следующей команды
```shell
./standalone.sh & exit
```

## Deployment

1. Скопировать war-архив в <wildfly-path>/standalone/deployments
1. Открыть терминал с запущенным WildFly (если он запущен, если нет - запустить) и наблюдать за процессом развёртыванием лабы

### или...

1. Открыть админскую консоль в браузере (по-умолчанию располагается на порте 9990 (localhost:9990 в строке браузера))
1. Логинился ранее созданным пользователем
1. Переходим во вкладку Deployments и жмякаем на (+) и выбираем Upload Deployment

Развёрнутый labname.war при стандартной конфигурация доступен по адресу locahost:8080/labname

## Установка portbase

Portbase необходим, чтобы задать смещение для всех портов нашего сервера приложений, т.к. не вы одни используете helios,
некоторые порты могут быть заняты.
Например, если при стандартной конфигурации у нас порт для доступа к развёртным приложениям - 8080, а для доступа к админской консоли - 9990, 
то при portbase=10 эти порты будут, соотвественно, равны 8080+10=8090 и 9990+10=10000

1. Откройте файл \<wildfly-directory\>/standalone/configuration/standalone.xml
1. Найдите строчку 
```xml
<socket-binding-group name="standard-sockets" default-interface="public" port-offset="${jboss.socket.binding.port-offset:0}">
```
1. Измените значение в соотвествии с вашим portbase
```
port-offset="${jboss.socket.binding.port-offset:0}" 
  
// port-offset меняем на ваш portbase, например, если у нас portbase=1111, то изменяем так 
port-offset="${jboss.socket.binding.port-offset:1111}"
```

## Что делать если не создаются таблицы в БД?

Вероятнее всего, приложение подключается к дефолтному датасорсу на WildFly. Найди следующую строчку в \<wildfly-directory\>/standalone/configuration/standalone.xml и удалите её:
```xml
<default-bindings context-service="java:jboss/ee/concurrency/context/default" datasource="java:jboss/datasources/ExampleDS" managed-executor-service="java:jboss/ee/concurrency/executor/default" managed-scheduled-executor-service="java:jboss/ee/concurrency/scheduler/default" managed-thread-factory="java:jboss/ee/concurrency/factory/default"/>
```