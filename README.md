# Django site

Докеризированный сайт на Django для экспериментов с Kubernetes.

Внутри конейнера Django запускается с помощью Nginx Unit, не путать с Nginx. Сервер Nginx Unit выполняет сразу две функции: как веб-сервер он раздаёт файлы статики и медиа, а в роли сервера-приложений он запускает Python и Django. Таким образом Nginx Unit заменяет собой связку из двух сервисов Nginx и Gunicorn/uWSGI. [Подробнее про Nginx Unit](https://unit.nginx.org/).

## Как запустить dev-версию

Запустите базу данных и сайт:

```shell-session
$ docker-compose up
```

В новом терминале не выключая сайт запустите команды для настройки базы данных:

```shell-session
$ docker-compose run web ./manage.py migrate  # создаём/обновляем таблицы в БД
$ docker-compose run web ./manage.py createsuperuser
```

Для тонкой настройки Docker Compose используйте переменные окружения. Их названия отличаются от тех, что задаёт docker-образа, сделано это чтобы избежать конфликта имён. Внутри docker-compose.yaml настраиваются сразу несколько образов, у каждого свои переменные окружения, и поэтому их названия могут случайно пересечься. Чтобы не было конфликтов к названиям переменных окружения добавлены префиксы по названию сервиса. Список доступных переменных можно найти внутри файла [`docker-compose.yml`](./docker-compose.yml).

## Переменные окружения

Образ с Django считывает настройки из переменных окружения:

`SECRET_KEY` -- обязательная секретная настройка Django. Это соль для генерации хэшей. Значение может быть любым, важно лишь, чтобы оно никому не было известно. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#secret-key).

`DEBUG` -- настройка Django для включения отладочного режима. Принимает значения `TRUE` или `FALSE`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-DEBUG).

`ALLOWED_HOSTS` -- настройка Django со списком разрешённых адресов. Если запрос прилетит на другой адрес, то сайт ответит ошибкой 400. Можно перечислить несколько адресов через запятую, например `127.0.0.1,192.168.0.1,site.test`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#allowed-hosts).

`DATABASE_URL` -- адрес для подключения к базе данных PostgreSQL. Другие СУБД сайт не поддерживает. [Формат записи](https://github.com/jacobian/dj-database-url#url-schema).


# Внимание:
## Урок 1. Разверните сайт в Minikube выполнялся на МакОС, что принесло много нюансоов и описание сделано только для этого случая.

# Запуск проекта в minikube (локально)
[Kubectl](https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/) и [minikube](https://minikube.sigs.k8s.io/docs/) должны быть установлены и настроены.

В связи с тем, что VirtualBox (больше не поддерживается macOS) и Docker (не поддерживаает ingress в macOS) драйвер QEMU был выбран тут [minikube документация](https://minikube.sigs.k8s.io/docs/drivers/). Очень ваажно при установке выполнить [пункт Networking](https://minikube.sigs.k8s.io/docs/drivers/qemu/)

Запускаем minikube командой, важно указать драйвер qemu2 - уточню именно с "2" на конце
 
```sh
minikube start --driver qemu2 --network builtin --network=socket_vmnet
```

Затем запускаем 
```sh
minikube addons enable ingress
```
Который может попросить запустить дополнительные сервисы, запускаем их тоже

# Важно:
## Для тех, кто пойдет моим путем я выкладываю в репозитории файлы с моми паролями и настройками, это не правильно, НО СЪЭКОНОМИТ ВАМ КУЧУ ВРЕМЕНИ!

Итак если предыдущие шаги прошли без ошибок и у вас запущен Dockers-desktop смотрим IP вашего мака и прописываем его в docker-compose.yml который лежит в папке kubernetes. В моем случае там много IP шников, так как я работал с бука в разных местах - естественно выбирал текущий у себя. 

```sh
docker-compose up
```
Оставляем окно работающим. В другом окне запускаем
```sh
minikube  ip    
```
Командаа дает нужный IP который и прописываем в hosts, важно с правами администратора 

```sh
sudo nano /etc/hosts
```

строка например:
```
192.168.105.2   star-burger.test
```

Прописываем свои настройки в django-app-config.yml и заапускаем

```sh
kubectl apply -f django-app-config.yml
```

Образ джанги у меня лежит на https://hub.docker.com/ - вам надо создать свой для дальнейшей работы, затем запускаем

```sh
kubectl apply -f django-app-ingress.yaml
```

Чтобы посмотреть, что все прошло успешно пишем в браузере 
```
star-burger.test
```
## Важные замечания
Для того, чтобы посмотреть что вы сделали удобно воспользоваться командой
```sh
minikube dashboard 
```
Но надо иметь ввиду, что изменения и перезапуски в командной строке не всегда "доходят" до деплоя - я рекомендую в вебинтерфейсе почаще пользоваться кнопкой рестарт. 
<img width="1119" alt="image" src="https://github.com/SGKespace/k8s-test-django/assets/55636018/1ed6de48-3230-4ee4-bf85-fd35fa088621">
<img width="871" alt="image" src="https://github.com/SGKespace/k8s-test-django/assets/55636018/98202f10-0737-4555-92c3-72f39300614c">

Напоследок сделаем важные приемы работы с нашим сайтом на джанге

Например миграции Django к базе данных:
```sh
kubectl apply -f kubernetes/django-migrate.yaml
```

Или запланируем задачу, удаления сессий Django:
```sh
kubectl apply -f kubernetes/django-clearsessions.yaml
```

## Сайт в кластере Yandex Cloud

Получите данные для 2 урока
<img width="925" alt="image" src="https://github.com/SGKespace/k8s-test-django/assets/55636018/c5de9228-e59a-4300-b882-99704181b6cf">

###  Запустите графическую оболочку Lens Desktop и посмотрите выделенные для вас параметры базы данных
<img width="1279" alt="image" src="https://github.com/SGKespace/k8s-test-django/assets/55636018/abdcd6ad-3b3e-4e43-b09f-e3bbe05c3235">
нас интересуют эти секреты
<img width="1205" alt="image" src="https://github.com/SGKespace/k8s-test-django/assets/55636018/bac37183-465c-4b13-995f-9b293276beb2">
Создаем файл django-secret.yml для DATABASE_URL используем шаблон POSTGRES_URL - postgres://<пользователь postgres>:<пароль пользователя>@<хост базы данных>:<порт бд>/<имя бд> Обратите внимание данные пишутся в формате  Opaque

 ```
apiVersion: v1
kind: Secret
metadata:
  name: django-secret
  namespace: edu-jovial-northcutt
  labels:
    app: django-app
type: Opaque
data:
  SECRET_KEY: QTBbQnQiVjU5LnhBLWJLT3wvQSIiL1ouISNZOndmcFI=
  DATABASE_URL: cG9zdGdyZXM6Ly9lZHUtam92aWFsLW5vcnRoY3V0dDp3YlFXSkQzdVZAbFEzQWJXQHJjMWItajAzYTIxOGtob3hqbHdkaS5tZGIueWFuZGV4Y2xvdWQubmV0OjY0MzIvZWR1LWpvdmlhbC1ub3J0aGN1dHQ/c3NsbW9kZT1yZXF1aXJl
```

Внесите исправления в манифесты в соответсвии с выданным вам заданием и  запустите их подставив свои значения пода и ппространства имен

```shell
kubectl apply --filename django-app-config.yml --namespace=edu-jovial-northcutt
kubectl apply --filename django-secret.yml --namespace=edu-jovial-northcutt
kubectl apply --filename django-app-deployment.yml  --namespace=edu-jovial-northcutt
```

Зайдите внутрь контейнераа, сделайте там миграции и после этого создайте суперпользователя (исправьте на свой под и  пространство имен)

```shell
kubectl exec  -ti django-app-deployment-6498754757-d8vng  --namespace=edu-jovial-northcutt sh
```

### Перейдите по ссылке которую вам выдали для просморта результата и авторизуйтесь под суперпользователем

<img width="618" alt="image" src="https://github.com/SGKespace/k8s-test-django/assets/55636018/e5597f23-b646-4f61-9a5a-cb95f4e60ae8">

[Ссылка на выделенный мне ресурс](https://sirius-env-registry.website.yandexcloud.net/edu-jovial-northcutt.html)
[Ссылка на сайт](https://edu-jovial-northcutt.sirius-k8s.dvmn.org/admin/)
