# Описание задания
Требуется разработать два микросервиса на языке программирования Golang: _Order_ и _Account_. 
У сервиса _Order_ существует единственный бизнес-сценарий под названием _CreateOrder_. 

При создании заказа необходимо выполнить следующее условие: 
проверить, достаточно ли у пользователя, создающего заказ, 
средств на его счете в сервисе _Account_ для осуществления операции. 
Если средств достаточно, то необходимо уменьшить их количество в сервисе _Account_, 
создав при этом заказ в сервисе _Order_.

Создание заказа необходимо реализовать асинхронно. 
После вызова метода _CreateOrder_ будет возвращен _OrderID_, 
по которому можно получить информацию о заказе методом _GetOrder_.

## Требования к реализации:
1. Сервисы _Order_ и _Account_ используют разные базы данных, предпочтительнее всего использовать PostgreSQL.

2. Обработаны случаи несогласованности данных в случае сбоя одного из сервисов и прерывания операции создания заказа.

3. Для взаимодействия между сервисами стоит выбрать один из предложенных брокеров на усмотрение: Kafka, RabbitMQ/ZeroMQ, Nats.

4. Обработчик запросов реализовать используя gRPC.

5. Учесть возможность поступления одновременных запросов на создания нескольких заказов

6. Операция создания заказа должна быть либо выполнена полностью, либо отклонена, исключая следующие сценарии:
    1. Заказ был создан, но средства пользователя не списаны.

    2. Заказ не был создан, но средства пользователя списаны.

    3. Заказ был создан, когда у пользователя не было достаточного количества средств на счете.

7. Каждый сервис должен быть упакован в Dockerfile и связан с другими через docker-compose.


# Архитектура
Для решения данной задачи была выбрана event-driven архитектура межсервисного взаимодействия.

В частности, был выбран паттерн Outbox, обеспечивающий надёжное фиксирование и хранение данных,
ставя в качестве источника правды БД, перекладывая обязанность на непосредственную межсервисную передачу
на брокера (в данном случае, Kafka).

В качестве CDC-решения, обеспечивающего надёжный перехват и отправку событий между сервисами,
были выбраны Kafka Connect и Debezium.


# Запуск

## Запуск всех сервисов
```shell
make configure # Создаёт .env файл с данными из .env.example
make up-services
```

## Запуск тестов
Запуск всех тестов:
```shell
make test
```

Запуск только unit-тестов:
```shell
make test-unit
```

Запуск только E2E-тестов:
```shell
make test-e2e
```

## Запуск форматтеров и линтеров
Требования: установленные [golangci-lint](https://github.com/golangci/golangci-lint) и [gofumpt](https://github.com/mvdan/gofumpt)

Запуск и форматтеров, и линтеров:
```shell
make lint
```

Только форматтера:
```shell
make lint-format
```

Только линтеров:
```shell
make lint-check
```