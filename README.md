# Load tests for Rocket.Chat

This repository implements a load test for [Rocket.Chat](https://github.com/RocketChat).

# Тест Rocket.Chat

Для тестирования данного скрипта необходимо иметь серверную часть с установленным RocketChat’ом и вторую машину (далее тестовый стенд) с которой будет производиться нагрузочное тестирование.

# Серверная часть.

Для тестирования RocketChat, предполагается, что серверная часть используется на ОС Server Ubutu 16.10, и установлены: RocketChat, MongoDB, Git и Node.js.

Дополнительные приложения на серверной части: APG (automatic password generator) – генератор паролей, используемый для генерации случайных имён пользователей;

## Последовательность действий на серверной части:

a)    Устанавливаем APG.

sudo apt-get install apg

b)    Клонируем скрипт с github и переходим в его корневую директорию:

- git clone https://github.com/Familiar89/rocketchat_loadtest_message_listener.git

cd /home/user/rocketchat_loadtest_message_listener

c)    Устанавливаем необходимые пакеты командой:

npm install

d)    Генерируем пользователей (1000 пользователь):

./message_listener/lib/generate-users.sh 1000

создаются 2 файла (user_insert_script.js и userdata.js)

e)    Импортируем созданных пользователей в базу RocketChat:

mongo 127.0.0.1/rocketchat ./message_listener/lib/user_insert_script.js

f)    Файлы с пользователями (user_insert_script.js и userdata.js) загружаем на тестовый стенд с которого будем запускать тест.

# Тестовый стенд.

Запуск скрипта на тестовом стенде предполагает установленную ОС Ubutu 16.10 или Window OS.

## Дополнительные приложения:

Node.js – для запуска js файлов.

Python-pip – для запуска python файла, выводящего результаты;

Numpy – так же для запуска python файла, выводящего результаты.

## Последовательность действий на тестовом стенде:

a)    Устанавливаем дополнительное ПО: Node.js, Python-pip.

sudo apt install curl

curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash –

sudo apt install nodejs

apt-get install python-pip

b)    Клонируем скрипт с github

git clone https://github.com/Familiar89/rocketchat_loadtest_message_listener.git

c)    Переходим в корневую директорию скаченного скрипта:

cd /home/user/rocketchat_loadtest_message_listener

d)    Устанавливаем необходимые пакеты:

- npm install

- pip install numpy

e)    Копируем файл user_insert_script.js, который загрузили с сервера в папку ~/message_listener/lib

f)    Выполняем команду:

node message_listener/app.js 0 -i 0.1 -j 0.05 -n 100 -u 1000 -w 5 –s http://your-host-name.com-as-accessed-from-internet/

Этой командой мы запускаем 1000 клиентов (500 получателей, 500 отправителей). 

Пользователь-отправитель отправляет 100 сообщений с интервалом 0,1 ± 0,05 секунды. 

Пользователь-получатель будет ждать сообщения, которые должны прийти, до 5 секунд.

Где: http://your-host-name.com-as-accessed-from-internet/ - ссылка на сервер RocketChat.

## Использование скрипта: apps.js

OFFSET [options]

--help, -h

--message-interval VALUE, -i VALUE

--message-interval-jitter VALUE, -j VALUE

--message-count VALUE, -n VALUE

--server-url VALUE, -s VALUE

--receiver-additional-waiting-time VALUE, -w VALUE

--user-count VALUE, -u VALUE

## Описание настроек скрипта.

OFFSET:

Смещение пользователя (из файла lib/userdata.js), полезно при запуске нескольких экземпляров с одним и тем же набором данных (смещение на N).

--message-interval, -i:

Интервал между сообщениями в секундах. По умолчанию равен 1 секунде.

--message-interval-jitter, -j:

Случайный джиттер для добавления/вычитания относительно интервала сообщения. По умолчанию равен 0.5 секунды.

--message-count, -n:

Количество сообщений, которые должен отправить пользователь-отправитель. Значение по умолчанию: 10 сообщений.

--server-url, -s:

URL-адрес RocketChat сервера. По умолчанию задан сервер https://open.rocket.chat/. Обратите внимание, что если для доступа к чату у вас в конце адреса дописывается порт, то сервер задается в формате http://host:port/

--receiver-additional-waiting-time, -w:

Время ожидания сообщений (в секундах) после прекращения действий отправителем. По умолчанию: 1 секунда.

--user-count, -u:

Количество клиентов, участвующих в тестировании, должно быть кратно двум, т.к. для каждого отправителя должен быть получатель. Значение по умолчанию: 2 пользователя. Значения по умолчанию можно изменить в файле app.js

g)    После завершения выполнения скрипта, формируется файл с данными которые мы видели в консоли.

h)    Запускаем python файл:

parse_outcome.py <filename>,
  
filename  - файл сформированный после завершения выполнения js скрипта.
