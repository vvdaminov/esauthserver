# Разное

## Настройка среды

В качестве базового выбран стек Ubuntu 19.04 + PostgreSQL + Nginx + Gunicorn + Python 3.0 + Django 2

### Установка

    $ sudo apt update && sudo apt upgrade
    $ sudo apt install python3-pip python3-dev libpq-dev curl
    $ sudo sudo ln -s /usr/bin/pip3 /usr/bin/pip

### nginx

    $ sudo apt install nginx

Управление nginx

    $ sudo service nginx start
    $ sudo service nginx stop
    $ sudo service nginx restart 
    $ sudo systemctl restart nginx      
    $ sudo nginx -t #тест

Файл etc/hosts добавляем

    127.0.0.1	esauthserver.ru
    127.0.0.1	domsoviet.ru

Файл /etc/nginx/sites-available/esauthserver.ru

    server {
        listen 80;
        server_name esauthserver.ru www.esauthserver.ru;

        location = /favicon.ico { access_log off; log_not_found off; }
        location /static/ {
            root /home/vlad/dev/venv/esauthserver;
        }

        location / {
            root /home/vlad/dev/venv/esauthserver;
                include proxy_params;
                proxy_pass http://unix:/run/gunicorn.sock;
        }
    }    
Аналогично файл /etc/nginx/sites-available/domsoviet.ru

Активация настроек сайтов

    $ sudo ln -s /etc/nginx/sites-available/esauthserver.ru /etc/nginx/sites-enabled
    $ sudo ln -s /etc/nginx/sites-available/domsoviet.ru /etc/nginx/sites-enabled
    $ sudo nginx -t #тест
    $ sudo systemctl restart nginx      
    
### postgresql

    $ sudo apt-get install postgresql-server-dev-11                        #(?нужно ли)
    $ sudo apt-get install postgresql postgresql-client postgresql-common

    $ sudo -u postgres psql
    p=# CREATE DATABASE ESAUTHSERVER;
    p=# CREATE DATABASE DOMSOVIET;
    p=# CREATE USER admin WITH PASSWORD 'qwer1234';
    p=# ALTER ROLE admin SET client_encoding TO 'utf8';
    p=# ALTER ROLE admin SET default_transaction_isolation TO 'read committed';
    p=# GRANT ALL PRIVILEGES ON DATABASE ESAUTHSERVER TO "admin";
    p=# GRANT ALL PRIVILEGES ON DATABASE DOMSOVIET TO "admin";
    p=# \q

ссылка на пример-источник:
https://www.8host.com/blog/razrabotka-django-prilozheniya-na-postgresql-nginx-gunicorn-v-ubuntu-18-04/

### Виртуальное окружение

Установка    

    $ sudo apt install python3-venv

Переходим в vlad@devstation:~/dev/venv (может быть другой путь)
Экземпляр вирутального окружения myenv (может иметь другое название)
  
    $ python3 -m venv myenv

создалась структура папок ~/dev/venv/myenv/.

    $ source ./myenv/bin/activate                                                # активация виртуального окружения (deactivate - деактивация)
    (myenv) vlad@devstation:~/dev/venv$ pip install django                       # инстраляция django

Создание проектов 

    {Ё$} это путь {"~/dev/venv$"})
    
    (myenv):~$ django-admin startproject esauthserver                              # название проекта сервера авторизациии и хранения данных 
    (myenv):~$ django-admin startproject domsoviet                                 # проект сайта-клиента
                                                                                   # (проекты создаются в отдельной папке)
    
    (myenv):~$ pip install psycopg2                                                # адаптер PostgreSQL для Python  

    (myenv):~$ pip install gunicorn                                                # gunicorn


(!)запуск в папках проектов

    (myenv):~/esauthserver$ gunicorn --bind 0.0.0.0:8001 esauthserver.wsgi         # запуск
    (myenv):~/domsoviet$ gunicorn --bind 0.0.0.0:8002 domsoviet.wsgi               # запуск
    (myenv):~$ deactivate                                                          # остановка при необходимости
    
Посмотреть процессы gunicorn и убить их
    
    (myenv):~$ ps ax|grep gunicorn
    (myenv):~$ kill 18765 или                                                 # не срабатывает или что-то делаю не так
    (myenv):~$ killall gunicorn                                               # поэтому убиваю все процессы 

Результат - сайты вызываются по адресам

    http://esauthserver.ru:8001/
    http://esauthserver.ru:8001/admin/
    http://domsoviet.ru:8002/

## esauthserver

Редактирование settings.py (по аналогии для domsoviet.ru, уже есть на github)

    "-":
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
        }
    }
    "+":
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql_psycopg2',
            'NAME': 'esauthserver',
            'USER': 'admin',
            'PASSWORD': 'qwer1234',
            'HOST': '127.0.0.1',
            'PORT': 5432,
        }
    }

    """
    ALLOWED_HOSTS = ['esauthserver.ru']
    """

    """
    LANGUAGE_CODE = 'ru-ru'

    TIME_ZONE = 'Asia/Novosibirsk'
    """

    """
    STATIC_URL = '/stati
    STATIC_ROOT = os.path.join(BASE_DIR, 'static/')
    """

Теперь нужно переместить исходную схему базы данных в базу данных PostgreSQL:

    (myenv):~/esauthserver$ ./manage.py makemigrations
    (myenv):~/esauthserver$ ./manage.py migrate

Создание администратора проекта (почту указывать не обязательно):

    (myenv):~/esauthserver$ ./manage.py createsuperuser

Переместите весь статический контент в подготовленный каталог:

    (myenv):~/esauthserver$ ./manage.py collectstatic    


### django-oauth-toolkit

Далее сервера осуществлялась по руководству:
https://django-oauth-toolkit.readthedocs.io/en/latest/tutorial/tutorial_01.html

до тех пункта

    Include the required hidden input in your login template, registration/login.html. The {{ next }} template context variable will be populated with the correct redirect value. See the Django documentation for details on using login templates.

    <input type="hidden" name="next" value="{{ next }}" />

не ясно, где должен быть файл registration/login.html и что из себя должен представлять готовая базовая структура.

