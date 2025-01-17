## Тестовое задание

Есть несколько рабочих сервисов, у каждого сервиса есть состояние работает/не работает/работает нестабильно.

Требуется написать API который:

1. Получает и сохраняет данные: имя, состояние, описание
2. Выводит список сервисов с актуальным состоянием
3. По имени сервиса выдает историю изменения состояния и все данные по каждому состоянию

Дополнительным плюсом будет

1. По указанному интервалу выдается информация о том сколько не работал сервис и считать SLA в процентах до 3-й запятой

Вывод всех данных должен быть в формате JSON

## Решение

### Реализация

[![Maintainability](https://api.codeclimate.com/v1/badges/b0163fa1df13db99f77b/maintainability)](https://codeclimate.com/github/rezajkee/task-python/maintainability)
[![Linters](https://github.com/rezajkee/task-python/actions/workflows/linters.yml/badge.svg?branch=solution)](https://github.com/rezajkee/task-python/actions/workflows/linters.yml)

Под капотом у сервиса фреймворк FastAPI и ORM SQLAlchemy, в качестве базы данных по умолчанию – PostgreSQL.

Реализована аутентификация через JSON Web Token, срок действия токена – 120 минут. Доступ к функциям сервиса предоставляется
только аутентифицированным пользователям с правами "сотрудник" или "админ". По умолчанию зарегистрированный пользователь
имеет права уровня "пользователь". Для создания администратора можно использовать python-скрипт. Администратор, в свою очередь,
может менять права и удалять любых пользователей.

Также реализованы базовые CRUD операции для сервисов и их состояний (create, update и delete доступны только админам).
При добавлении статуса сервиса временной отпечаток проставляется автоматически и не может быть изменён средствами редактирования.

Добавлен вывод списка сервисов с их актуальным состоянием.

Реализован вывод истории состояний по имени сервиса с возможностью пагинации (параметры запроса limit (кол-во записей)
и offset (смещение)).

Добавлен вывод времени простоя/нестабильной работы сервиса и его SLA в процентах за указанный период. Начальная и конечная даты
указываются параметрами запроса в формате "dd.mm.yy_HH.MM", а также можно использовать "now" для конечной даты. Параметры по
умолчанию "31.01.23_00.00" — "now".

### Установка

Для установки требуется Python 3.8.1 и выше.

1. Клонировать репозиторий удобным способом, перейти в директорию с проектом

    ```
    git clone https://github.com/rezajkee/task-python.git
    cd task-python
    ```

2. Установить зависимости

    ```
    pip install -r requirements.txt
    ```

    Для установки через poetry:

    ```
    poetry install
    ```

3. Создать в корне проекта файл `.env` с переменными для подключения к своей БД
(её нужно создать самостоятельно) и JWT_SECRET. Файл для примера: `.env.sample`

4. Запустить миграции 

    ```
    alembic upgrade head
    ```

5. Для создания администратора через консоль нужно добавить корень проекта в PYTHONPATH

    ```
    export PYTHONPATH=./
    ```

    И запустить скрипт

    ```
    python commands/create_admin.py -e ваш_email -p ваш_пароль -f ваше_имя -l ваша_фамилия
    ```

6. Запустить сервер

    ```
    uvicorn main:app
    ```

### Запуск в контейнере с помощью Docker Compose

1. Клонировать репозиторий удобным способом, перейти в директорию с проектом

    ```
    git clone https://github.com/rezajkee/task-python.git
    cd task-python
    ```

2. Создать в корне проекта файл `.env` с переменными для подключения к БД
и JWT_SECRET. Файл для примера: `.env.sample`

3. Запустить контейнер

    ```
    docker-compose up -d
    ```

4. Запустить миграции 

    ```
    docker-compose exec web alembic upgrade head
    ```

5. Создать пользователя с правами администратора

    ```
    docker-compose exec web python commands/create_admin.py -e ваш_email -p ваш_пароль -f ваше_имя -l ваша_фамилия
    ```

### API Endpoints

На этой странице находится интерактивная документация по API (предоставляемая Swagger UI):

```
http://localhost:8000/docs
```

| HTTP метод | Эндпоинт | Действие |
| --- | --- | --- |
| POST | /register | Регистрация с правами "пользователь", содержит токен в теле ответа |
| POST | /login | Вход в систему, содержит токен в теле ответа |
| GET | /users | Вывод списка всех пользователей, при передаче в параметре запроса email — находит нужного юзера |
| DELETE | /users/{user_id} | Удаление пользователя |
| PUT | /users/{user_id}/make-admin | Смена прав юзера на администратора |
| PUT | /users/{user_id}/make-staff | Смена прав юзера на сотрудника |
| GET | /services | Вывод списка всех сервисов в базе данных |
| POST | /services | Добавление сервиса в базу данных |
| PUT | /services/{service_id} | Обновление данных сервиса |
| DELETE | /services/{service_id} | Удаление сервиса и всех его состояний |
| GET | /services/states | Вывод списка сервисов с актуальным состоянием |
| GET | /services/states/{service_name} | Вывод истории состояний сервиса, доступна пагинация в параметрах запроса |
| GET | /services/states/{service_name}/sla | Вывод даунтайма и SLA сервиса, интервал указывается в параметрах запроса |
| GET | /states | Вывод списка всех состояний в базе данных, доступна пагинация в параметрах запроса |
| POST | /states | Добавление состояния в базу данных, время добавляется автоматически |
| PUT | /states | Обновление данных состояния, время не редактируется |
| DELETE | /states | Удаление состояния из базы данных |

 Все `GET` запросы доступны пользователям с правами "сотрудник" и "админиcтратор".
 `POST`, `PUT` и `DELETE` запросы, не считая первые два эндпоинта для аутентификации,
 доступны только администраторам.
