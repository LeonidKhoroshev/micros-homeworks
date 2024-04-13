
# Домашнее задание к занятию «Микросервисы: принципы» - Леонид Хорошев

Вы работаете в крупной компании, которая строит систему на основе микросервисной архитектуры.
Вам как DevOps-специалисту необходимо выдвинуть предложение по организации инфраструктуры для разработки и эксплуатации.

## Задача 1: API Gateway 

Предложите решение для обеспечения реализации API Gateway. Составьте сравнительную таблицу возможностей различных программных решений. На основе таблицы сделайте выбор решения.

Решение должно соответствовать следующим требованиям:
- маршрутизация запросов к нужному сервису на основе конфигурации,
- возможность проверки аутентификационной информации в запросах,
- обеспечение терминации HTTPS.

| API Gateway | Маршрутизация запросов к нужному сервису на основе конфигурации | Возможность проверки аутентификационной информации в запросах | Обеспечение терминации HTTPS |
|-------------|-----------------------------------------------------------------|---------------------------------------------------------------|------------------------------|
| Kong        | + | + | + |
| Tyk | + | + | + |
| Yandex API Gateway | + | + | + |
| Kubernetes API Gateway| + | + | + |

В целом вышеперечисленные решения довольно схожи по функционалу, поэтому выбор любого из них не будет считаться ошибкой. Для работы небольших сервисов я бы рекомендовал решение от Yandex, помимо исчерпывающей документации на русском языке и простоты настройки он позволяет существенно "упростить жизнь", особенно, если планируется использовать какие-нибудь сервисы Яндекса (карты, погода, оплата). Однако, при росте проекта и его масштабировании использование Yandex API Gateway, как собственно и API прочих облачных провайдеров (подобные решения существуют у Amazon, Azure и многих других) может привести к росту затрат, так как работа API тарифицируется платформами, предосьтавляющими подобные решения. В качестве выхода из данной ситуации рекомендуется использовать Kong (по информации из открытых источников - самое популярное API решение в мире). Он может работать на любой платформе, поддерживает гибридную и мультиоблачную инфраструктуру, а также оптимизирован для микросервисов и распределенных архитектур. Tyk мощный, легкий и полнофункциональный API-шлюз с открытым исходным кодом, написанный с нуля с использованием языка программирования Go. Kong - лучший вариант с точки зрения максимальной производительности, в то время как Tyk обеспечивает хорошую производительность с прицелом на простоту эксплуатации.

Принимая во внимание то, что наш проект будет достаточно большой либо создается с прицелом на дальнейшее развитие и кратный рост, оптимальное решение - Kong. Оно позводит нам не упереться в технологические ограничения и избежать излишних финансовых и ресурсных затрат.

## Задача 2: Брокер сообщений

Составьте таблицу возможностей различных брокеров сообщений. На основе таблицы сделайте обоснованный выбор решения.

Решение должно соответствовать следующим требованиям:
- поддержка кластеризации для обеспечения надёжности,
- хранение сообщений на диске в процессе доставки,
- высокая скорость работы,
- поддержка различных форматов сообщений,
- разделение прав доступа к различным потокам сообщений,
- простота эксплуатации.

| Брокер сообщений | Поддержка кластеризации | Хранение сообщений на диске в процессе доставки | Высокая скорость работы | Поддержка различных форматов сообщений | Простота эксплуатации |
|------------------|-------------------------|-------------------------------------------------|-------------------------|----------------------------------------|-----------------------|
| Kafka            |           +             |                      +                          | - (см. примечание 1)    |       - (см. примечание 4 )            | +/- (см. примечание 5)|
| RabbitMQ         | +/- (см. примечание 2)  |               - (см. примечание 3)              |            +            |                   +                    |        +              |

Примечание 1. Потенциальная разбалансированность нагрузки между разными консьюмерами и более высокая задержка обработки данных, относительно RabbitMQ

Примечание 2. Сложности при горизонтальном масштабировании в кластере. Необходимо добавлять настройки кластеризации над очередями и выбирать, чем пожертвовать — гибкостью или скоростью.

Примечание 3. После получения консьюмерами сообщение удаляется из очереди Благодаря этому одно и то же сообщение может быть обработано только одним консьюмером и не хранится дольше необходимого.

Примечание 4. В работе используется только собственный двоичный протокол поверх TCP.

Примечание 5. Для эксплуатации Kafka требуются знания о распределенных системах, параллельной обработке, конфигурациях и мониторинге кластера, а также знание проткола Kafka. RabbitMQ в свою очередь более прост с точки зрения настройки и администрирования, а также обладает удобной админкой. 
Окончательное решение необходимо принимать, имея чуть больше информации о проекте. Kafka больше подходит для обработки больших объёмов данных, сотен тысяч сообщений в секунду, которые читаются тысячами подписчиков (как пример - банкинг, телеком социальные сети, интернет вещей). RabbitMQ больше подходит для интеграции микросервисов, потоковой передачи данных в режиме реального времени или при передаче работы удалённым работника.

Возвращаясь к нашему проекту, с учетом того что на первом этапе в качестве API Gateway выбран Kong, то целесообразнее использовать Kafka, обеспечив их взаимодействие через плагин `Kafka Upstream`.


## Задача 3: API Gateway * (необязательная)

### Есть три сервиса:

**minio**
- хранит загруженные файлы в бакете images,
- S3 протокол,

**uploader**
- принимает файл, если картинка сжимает и загружает его в minio,
- POST /v1/upload,

**security**
- регистрация пользователя POST /v1/user,
- получение информации о пользователе GET /v1/user,
- логин пользователя POST /v1/token,
- проверка токена GET /v1/token/validation.

### Необходимо воспользоваться любым балансировщиком и сделать API Gateway:

**POST /v1/register**
1. Анонимный доступ.
2. Запрос направляется в сервис security `POST /v1/user`.

**POST /v1/token**
1. Анонимный доступ.
2. Запрос направляется в сервис security `POST /v1/token`.

**GET /v1/user**
1. Проверка токена. Токен ожидается в заголовке Authorization. Токен проверяется через вызов сервиса security `GET /v1/token/validation/`.
2. Запрос направляется в сервис security `GET /v1/user`.

**POST /v1/upload**
1. Проверка токена. Токен ожидается в заголовке Authorization. Токен проверяется через вызов сервиса security `GET /v1/token/validation/`.
2. Запрос направляется в сервис uploader `POST /v1/upload`.

**GET /v1/user/{image}**
1. Проверка токена. Токен ожидается в заголовке Authorization. Токен проверяется через вызов сервиса security `GET /v1/token/validation/`.
2. Запрос направляется в сервис minio `GET /images/{image}`.


### Ожидаемый результат

Результатом выполнения задачи должен быть docker compose файл, запустив который можно локально выполнить следующие команды с успешным результатом.
Предполагается, что для реализации API Gateway будет написан конфиг для NGinx или другого балансировщика нагрузки, который будет запущен как сервис через docker-compose и будет обеспечивать балансировку и проверку аутентификации входящих запросов.
Авторизация
```
curl -X POST -H 'Content-Type: application/json' -d '{"login":"bob", "password":"qwe123"}' http://localhost/token
```

**Загрузка файла**

```
curl -X POST -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJib2IifQ.hiMVLmssoTsy1MqbmIoviDeFPvo-nCd92d4UFiN2O2I' -H 'Content-Type: octet/stream' --data-binary @yourfilename.jpg http://localhost/upload
```

**Получение файла**
```
curl -X GET http://localhost/images/4e6df220-295e-4231-82bc-45e4b1484430.jpg
```

---

1. Выполнение данного задания необходимо начать с сохранения себе на ВМ  [дополнительных материалов](https://github.com/netology-code/devkub-homeworks/tree/main/11-microservices-02-principles)

```
mkdir micros
cd micros
git init
git pull https://github.com/LeonidKhoroshev/micros-homeworks
cd 11-microservices-02-principles
```

2. Поскольку по условиям предпочтительно использовать ngnix (как я понял у него один из самых низких порогов входа), ознакомимся с [документацией](https://www.nginx.com/blog/deploying-nginx-plus-as-an-api-gateway-part-1/)

3. Руководствуясь документацией из предыдущего пункта и рядом других [материалов](https://nginx.org/ru/) создаем файл [nginx.conf](https://github.com/LeonidKhoroshev/micros-homeworks/blob/main/11-microservices-02-principles/gateway/nginx.conf)

4. Пробуем запустить наши сервисы
```
docker-compose up
```

Сервис `Security` завершился с ошибкой
```
security-1       | Traceback (most recent call last):
security-1       |   File "/app/./server.py", line 2, in <module>
security-1       |     from flask import Flask, request, make_response, jsonify
security-1       |   File "/usr/local/lib/python3.9/site-packages/flask/__init__.py", line 14, in <module>
security-1       |     from jinja2 import escape
security-1       | ImportError: cannot import name 'escape' from 'jinja2' (/usr/local/lib/python3.9/site-packages/jinja2/__init__.py)
```

Для успешного запуска необходимо откорректировать [Dockerfile](https://github.com/LeonidKhoroshev/micros-homeworks/blob/main/11-microservices-02-principles/security/Dockerfile)
```
FROM python:3.10-alpine
WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt
COPY src ./

CMD [ "python", "./server.py" ]
```

и [requirements.txt](https://github.com/LeonidKhoroshev/micros-homeworks/blob/main/11-microservices-02-principles/security/requirements.txt),
```
Flask==3.0.2
passlib==1.7.4
PyJWT==2.0.1
prometheus-flask-exporter==0.23.0
```
указав более свежие версии пакетов, чем представленные в задании.

Сервис `gateway` также завершился с ошибкой
```
nginx: [emerg] unexpected "{" in /etc/nginx/nginx.conf:50
```

Корректируем указанный блок в docker-compose.yaml
```
        location ~ /v1/user/(.*) {
            set $image $1;
            proxy_pass http://minio/images/$image;
            proxy_pass_request_headers on;
            proxy_set_header Authorization $http_authorization;
```

Запускаем повторно
```
docker compose up
docker ps -a
```

![Alt_text](https://github.com/LeonidKhoroshev/micros-homeworks/blob/main/11-microservices-02-principles/screenshots/micros1.png)


5. Получаем токен
```
curl -X POST -H 'Content-Type: application/json' -d '{"login":"bob", "password":"qwe123"}' http://localhost/v1/token
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJib2IifQ.hiMVLmssoTsy1MqbmIoviDeFPvo-nCd92d4UFiN2O2I[root@microservices 11-microservices-02-principles]#
```
6. Загружаем картинку
```
curl -X POST \
  -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJib2IifQ.hiMVLmssoTsy1MqbmIoviDeFPvo-nCd92d4UFiN2O2I' \
  -H 'Content-Type: application/octet-stream' \
  --data-binary @1.jpg \
  http://localhost/v1/upload
```

Снова ошибка:
```
 <html>
<head><title>413 Request Entity Too Large</title></head>
<body>
<center><h1>413 Request Entity Too Large</h1></center>
<hr><center>nginx/1.25.4</center>
</body>
</html>
```

Как оказалось, в настройках `Nginx` по умолчанию максимальный размер файла - 1 Мб, это надо обязательно учитывать.

Повторно загружаем маленькую картинку:

![Alt_text](https://github.com/LeonidKhoroshev/micros-homeworks/blob/main/11-microservices-02-principles/screenshots/micros2.png)

7. Проверяем, что все наша картинка существует
```
curl localhost/user/1.jpg > 1.gpg
```

![Alt_text](https://github.com/LeonidKhoroshev/micros-homeworks/blob/main/11-microservices-02-principles/screenshots/micros3.png)


В качестве результата выпорлнения задания 3 направляю ссылки на откорректированные конфигурации файлов:

[docker-compose.yaml](https://github.com/LeonidKhoroshev/micros-homeworks/blob/main/11-microservices-02-principles/docker-compose.yaml)

[nginx.conf](https://github.com/LeonidKhoroshev/micros-homeworks/blob/main/11-microservices-02-principles/gateway/nginx.conf)

---
