# Отчет по лабораторной работе №3

# "Docker: сети, volumes, docker-compose"


## Цель работы

Научиться поднимать многоконтейнерный стек из трёх сервисов (nginx + Flask + PostgreSQL) через docker-compose, понимать взаимодействие контейнеров в сети и использовать persistent volumes.

# Docker networking

Для просмотра существующих сетей ввести команду `docker network ls` 

![подпись](/Students/K0409-24-1/Bordunova/images/photo34.png)

для просмотра детальной информации о bridge-сети ввести `docker network inspect bridge`, для создания изолированной bridge-сети выполнить `docker network create --driver bridge app-network`, для запуска контейнера PostgreSQL в созданной сети выполнить `docker run -d --name db --network app-network -e POSTGRES_PASSWORD=secret postgres:16-alpine`, для запуска тестового контейнера alpine в той же сети выполнить `docker run -it --rm --network app-network alpine sh`, внутри alpine для проверки связи с контейнером db ввести: `ping db` (убедиться, что DNS работает и контейнер виден по имени), для проверки доступности порта PostgreSQL ввести: nc -zv db 5432 (убедиться, что порт открыт), для выхода из контейнера alpine ввести: `exit`. Для проверки, что контейнер без пользовательской сети не видит db выполнить `docker run -it --rm alpine ping db`

Вывод: каждая сеть — отдельный namespace с собственной DNS

# Volumes и persistent data

Для создания volume выполнить команду `docker volume create pgdata`, для запуска контейнера PostgreSQL с подключением созданного volume выполнить `docker run -d --name postgres-persistent -e POSTGRES_DB=mydb -e POSTGRES_USER=user -e POSTGRES_PASSWORD=pass -v pgdata:/var/lib/postgresql/data postgres:16-alpine`

![подпись](/Students/K0409-24-1/Bordunova/images/photo35.png)

Для создания тестовых данных в базе внутри контейнера выполнить `docker exec -it postgres-persistent psql -U user -d mydb -c "CREATE TABLE items (id SERIAL, name TEXT); INSERT INTO items VALUES (1, 'test');"`

Для удаления контейнера выполнить `docker rm -f postgres-persistent`

Для повторного запуска нового контейнера с тем же volume выполнить `docker run -d --name postgres-restored -e POSTGRES_DB=mydb -e POSTGRES_USER=user -e POSTGRES_PASSWORD=pass -v pgdata:/var/lib/postgresql/data postgres:16-alpine`

# Docker-compose

Создать структуру проекта `mkdir -p ~/compose-lab/backend ~/compose-lab/frontend`. Написаны файлы: `backend/app.py` (Flask с подключением к БД), `backend/requirements.txt` (flask, psycopg2-binary), `backend/Dockerfile` (на базе python:3.12-alpine), `frontend/nginx.conf` (проксирование /api/ на backend). Создан `docker-compose.yml` с тремя сервисами `db — postgres:16-alpine` с `volume и healthcheck` через `pg_isready`, `backend` — сборка из `./backend`, переменные окружения для подключения к БД, `healthcheck` через `wget`, depends_on с ожиданием `healthy db`, `frontend` — `nginx:alpine` с пробросом порта `8080:80` и монтированием конфига

Подъём стека при помощи команд `docker compose up -d --build`. Проверка состояния: `docker compose ps`. Просмотр логов: `docker compose logs -f`

# Задания

Для получения первого результата, а именно вывода команды `docker compose ps` со статусом `healthy` для всех сервисов, выполнить сборку необходимо подождать примерно 10–15 секунд, пока каждый сервис пройдёт внутреннюю проверку здоровья, после чего ввести команду `docker compose ps`. В выводе в колонке `STATUS` у сервисов `db` и `backend` должно отображаться значение `healthy`

![подпись](/Students/K0409-24-1/Bordunova/images/photo5.png)

В терминале выполняется команда `curl localhost:8080/api/items`. В ответ должна вернуться JSON-строка с добавленными записями, например: `[{"id":1,"name":"apple"},{"id":2,"name":"banana"},{"id":3,"name":"cherry"}]`. Это подтверждает, что все три сервиса взаимодействуют корректно

![подпись](/Students/K0409-24-1/Bordunova/images/photo6.png)

Для получения третьего результата, то есть вывода команды `docker compose ps` после масштабирования сервиса `backend` до трёх экземпляров, необходимо, не останавливая работающий стек, выполнить команду `docker compose up -d --scale backend=3`. Docker Compose автоматически запустит два дополнительных контейнера для сервиса `backend`, сохраняя при этом уже работающий первый экземпляр. После завершения масштабирования следует ввести команду `docker compose ps`. В выводе будет видно, что теперь существуют три контейнера с именами, содержащими суффиксы `backend-1`, `backend-2` и `backend-3` (или аналогичные). Все три экземпляра должны находиться в статусе `running` или `healthy`, а общее количество контейнеров в стеке увеличится. Для проверки работоспособности можно повторно выполнить `curl localhost:8080/api/items` — приложение должно продолжать корректно отвечать, так как nginx балансирует нагрузку между тремя экземплярами backend. Таким образом, все три результата, требуемые преподавателем, будут получены.

![подпись](/Students/K0409-24-1/Bordunova/images/photo7.png)