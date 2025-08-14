# project2-zabbix
Вот подробная инструкция по установке Zabbix 7.0 на Ubuntu 24.04 

# Установка Zabbix 7.0 на Ubuntu 24.04

#### 1. Подготовка системы
##### Начните с обновления пакетов:

```bash
sudo apt update && sudo apt upgrade -y
```
#### 2. Установка репозитория Zabbix 7.0 LTS
##### Скачайте и установите пакет репозитория:


```Копировать код 
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu24.04_all.deb 
 sudo dpkg -i zabbix-release_7.0-1+ubuntu24.04_all.deb
 sudo apt update

```
#### 3. Установка компонентов Zabbix
##### Установите сервер, веб-интерфейс и агент:



##### Копировать код ```
sudo apt install -y zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent```
#### 4. Настройка MariaDB
##### Запустите безопасную настройку:



##### Копировать код
```sudo mysql_secure_installation```
##### Ответьте на вопросы:

##### Включите unix_socket аутентификацию (Y)
##### Не меняйте пароль root (n)
##### Удалите анонимных пользователей (Y)
##### Удалите тестовую БД (Y)
#### 5. Создание базы данных
##### Вручную создайте БД и пользователя:

##### входим и создаем
```
 sudo mysql -u root -p
```
##### sql

``` Копировать код
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin; 
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'StrongPassword123';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;
EXIT;

```
#### 6. Импорт схемы
##### Импортируйте схему:



Копировать код```
sudo zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -u zabbix -p zabbix```

##### Тут нужно подождать - После нескольких попыток и пересоздания пользователя импорт завершился успешно (204 таблицы).

#### 7. Настройка PHP
##### Обнаружили, что файл /etc/php/8.3/apache2/php.ini был пуст. --- тут исправлял ошибку с php ( не было файла /etc/php/8.3/apache2/php.ini возможно из-за того, что не была установленна конкретная версия для zabbix -- пришлось установить и поменять на другую версию php 

```sudo apt install -y php php-{bcmath,bz2,cli,common,curl,gd,gmp,imagick,intl,ldap,mbstring,mysql,opcache,readline,redis,xml,zip} libapache2-mod-php```

``` проверка 
user1@zabbix-serv:~$ ```sudo mysql -u zabbix -p -e "USE zabbix; SHOW TABLES;" | wc -l```
Enter password: 
204
```sudo apt install -y libapache2-mod-php8.3 php8.3```

```sudo update-alternatives --config php```
``` There are 2 choices for the alternative php (providing /usr/bin/php).

  Selection    Path                  Priority   Status
------------------------------------------------------------
* 0            /usr/bin/php.default   100       auto mode
  1            /usr/bin/php.default   100       manual mode
  2            /usr/bin/php8.3        83        manual mode

Press <enter> to keep the current choice[*], or type selection number: 2
```



```sudo vim /etc/php/8.3/apache2/php.ini```

```ini


memory_limit = 128M
post_max_size = 16M
upload_max_filesize = 2M
max_execution_time = 300
date.timezone = Europe/Moscow
```
#### 8. Запуск сервисов
##### Активируйте и запустите службы:


```sudo systemctl restart zabbix-server zabbix-agent apache2```
```sudo systemctl enable zabbix-server zabbix-agent apache2```
#### 9. Завершение установки через веб-интерфейс
##### Откройте http://ваш-сервер/zabbix и пройдите шаги мастера:

##### Проверьте требования (все OK)
##### Укажите данные БД:
##### Пользователь: zabbix
##### Пароль: StrongPassword123
##### Оставьте настройки сервера по умолчанию
##### Войдите в систему:

##### Логин: Admin
##### Пароль: zabbix
#### 10. Смена пароля Admin
##### Через веб-интерфейс:

##### Нажмите на иконку профиля → User settings
##### Перейдите на вкладку Password
##### Введите:
##### Текущий пароль: zabbix
##### Новый пароль: ваш_новый_пароль
##### Нажмите Update
##### Особенности вашей установки
##### Проблемы с PHP решились ручной копией конфига
##### Требовалось несколько попыток импорта БД
#### Использовали:
##### Ubuntu 24.04
##### Zabbix 7.0 LTS
##### MariaDB с unix_socket-аутентификацией
<img width="2874" height="1714" alt="image" src="https://github.com/user-attachments/assets/da965994-c624-4c09-b210-27768fdf1118" />




### Простая настройка мониторинга Docker Compose и Zabbix Agent для новичка
#### 1. Установка Zabbix Agent на клиентский сервер (который будем мониторить)
``` bash
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu24.04_all.deb
sudo dpkg -i zabbix-release_7.0-1+ubuntu24.04_all.deb
sudo apt update
sudo apt install -y zabbix-agent2
```
#### 2. Настройка агента (на клиенте)
##### Откройте конфиг:

```bash
sudo nano /etc/zabbix/zabbix_agent2.conf
```
##### Измените:

```ini
Server=IP_вашего_Zabbix_сервера
ServerActive=IP_вашего_Zabbix_сервера
Hostname=client-hostname  # Уникальное имя для этого хоста
```
##### Перезапустите агент:

```bash
sudo systemctl restart zabbix-agent2
sudo systemctl enable zabbix-agent2
```
#### 3. Добавление хоста в Zabbix (на сервере)
##### Откройте веб-интерфейс: http://ваш_zabbix_сервер/zabbix

##### Перейдите: Configuration → Hosts → Create host

##### Заполните:

##### Host name: client-hostname (как в конфиге агента)

##### Groups: Linux servers

##### Interfaces: Добавьте IP клиента, порт 10050 (Agent)

##### Templates:

##### Добавьте: Linux by Zabbix agent

##### Добавьте: Docker by Zabbix agent 2 (встроен в Zabbix 7.0)

#### 4. Настройка мониторинга Docker Compose (самая простая версия)
##### На клиентском сервере дайте агенту доступ к Docker:

```bash
sudo usermod -aG docker zabbix
sudo systemctl restart zabbix-agent2
```
##### Проверьте, что агент видит Docker:

```bash
sudo -u zabbix docker ps
```
#### 5. Что будет мониториться автоматически
##### После привязки шаблона Docker by Zabbix agent 2 Zabbix будет собирать:

##### Общее количество контейнеров

##### Состояние каждого контейнера (running/exited)

##### Использование CPU/RAM/сети

##### Дисковое пространство

#### 6. Проверка работы
##### На сервере Zabbix:

##### Перейдите: Monitoring → Hosts

##### Выберите ваш хост

 ##### Проверьте вкладки Latest data и Dashboards


<img width="2528" height="1690" alt="image" src="https://github.com/user-attachments/assets/91a8f286-b2b9-491a-a485-af702cf71010" />

