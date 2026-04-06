# Отчет по лабораторной работе №2

# "Docker: образы, Dockerfile, запуск контейнеров"


## Цель работы

Научиться писать Dockerfile, собирать образы, запускать контейнеры с необходимыми параметрами, понимать архитектуру слоёв Union FS и использовать multistage build для оптимизации размера образов.


## Создание первого Dockerfile

В ходе лабораторной работы были освоены основные приёмы работы с Docker. На первом этапе создано простое Flask-приложение с файлами `app.py` и `requirements.txt` [созданы в ~/docker-lab]. Написан первоначальный Dockerfile, который использовал полный образ python:3.12, копировал все файлы и устанавливал зависимости. После сборки образа `myapp:bad docker build -t myapp:bad` и запуска контейнера `docker run -d -p 5000:5000 --name app-bad myapp:bad` 

Проверить размер созданного образа при помощи команды `docker images myapp`, его большой размер >1Гб объясняется наличием компиляторов и инструментов разработки, а также отсутствием очистки кэша

![подпись](/Students/K0409-24-1/Bordunova/images/photo1.png)

## Multistage build

Сорать образ с тегом good при помощи команды `docker build -t myapp:good` Размер образа myapp:good уменьшился до 450 МБ

![подпись](/Students/K0409-24-1/Bordunova/images/photo2.png)

после запустить контейнер с ограничениями `docker run -d -p 5001:5000 --name app-good --memory="128m" --cpus="0.5" --restart=unless-stopped myapp:good` и проверить ограничения `docker stats app-good` — просмотр потребления ресурсов и действующих лимитов

![подпись](/Students/K0409-24-1/Bordunova/images/photo4.png)

Исследовать `docker history myapp:good` — вывод истории слоев

![подпись](/Students/K0409-24-1/Bordunova/images/photo3.png)

После sudo dpkg -i dive_0.12.0_linux_amd64.deb — установка утилиты dive, dive myapp:good — визуальный анализ содержимого каждого слоя, docker inspect myapp:good | jq '.[0].RootFS' — детальная информация о слоях

Для публикации на Docker Hub `docker login` — вход в аккаунт, `docker tag myapp:good ВАШЕ_ИМЯ/flask-demo:v1.0` — тегирование образа, `docker push ВАШЕ_ИМЯ/flask-demo:v1.0` — загрузка в реестр

URL: https://hub.docker.com/repository/docker/alinaaaaaaaa777/flask-demo/general