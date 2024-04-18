
# Домашнее задание к занятию «Микросервисы: масштабирование» - Леонид Хорошев

Вы работаете в крупной компании, которая строит систему на основе микросервисной архитектуры.
Вам как DevOps-специалисту необходимо выдвинуть предложение по организации инфраструктуры для разработки и эксплуатации.

## Задача 1: Кластеризация

Предложите решение для обеспечения развёртывания, запуска и управления приложениями.
Решение может состоять из одного или нескольких программных продуктов и должно описывать способы и принципы их взаимодействия.

Решение должно соответствовать следующим требованиям:
- поддержка контейнеров;
- обеспечивать обнаружение сервисов и маршрутизацию запросов;
- обеспечивать возможность горизонтального масштабирования;
- обеспечивать возможность автоматического масштабирования;
- обеспечивать явное разделение ресурсов, доступных извне и внутри системы;
- обеспечивать возможность конфигурировать приложения с помощью переменных среды, в том числе с возможностью безопасного хранения чувствительных данных таких как пароли, ключи доступа, ключи шифрования и т.п.

Оптимальным решением видится обеспечение развертывания инфраструктуры на базе Kubernetes. С точки зрения поддержки контейнеров - это наиболее удачное решение, обеспечивающее более широкий функционал, чем к примеру [Docker Swarm](https://habr.com/ru/companies/selectel/articles/672666/), который впринципе тоже может быть использован на небольших проектах, не имеющих перспектив кратного роста. 

Для обнаружения сервисов и маршрутизации запросов используется DNS-сервер. При выборе развертывания инфраструктуры на базе Kubernetes вполне можно использовать [кластерный DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/). При построении инфраструктуры на базе облачных решений, например от Yandex cloud удобнее всего воспользоваться уже готовым решением по [интеграции](https://yandex.cloud/ru/docs/managed-kubernetes/tutorials/custom-dns) кластера k8s c используемой зоной DNS.

Возможность [горизонтального масштабирвоания](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) в k8s обеспечивается за счет автомасштабирования подов при развертывании или наборе реплик. Масштабирование строится за счет периодического запроса в API метрик ресурсов основных показателей (использование ЦП и памяти).

Автомасштабирование кластера отвечает за масштабирование узлов, чем позволяет автоматически увеличивать и уменьшать рабочие нагрузки в зависимости от использования ресурсов. Автомасштабирование зависит от возможностей поставщика облачной инфраструктуры (пример -[Yandex cloud](https://yandex.cloud/ru/docs/managed-kubernetes/operations/autoscale#ca)), а горизонтальное масштабирование одинаково работает независимо от провайдера.

Разделение ресурсов в кластере k8s реализовано за счет [пространства имен](https://kubernetes.io/ru/docs/concepts/overview/working-with-objects/namespaces/), то есть Kubernetes поддерживает несколько виртуальных кластеров в одном физическом кластере, а эти виртуальные кластеры и называются пространствами имен. Они позволяют изолировать друг от друга ресурсы, обеспечив им разные права доступа и разную доступность как извне, так и изнутри системы.

Kubernetes использует следующие переменные среды:
- имя хоста контейнера;
- имя модуля и пространство имён;
- пользовательские переменные среды (как из определения Pod, так и переменные среды, статически указанные в образе Docker);
- чувствительные данные (пароли, ключи, токены и т.д.) доступны к хранению в [vault](https://habr.com/ru/companies/rshb/articles/759816/) (данный механизм аналогичен хранению секретов в Teggaform и Ansible, рассмотренными ранее).


## Задача 2: Распределённый кеш * (необязательная)

Разработчикам вашей компании понадобился распределённый кеш для организации хранения временной информации по сессиям пользователей.
Вам необходимо построить Redis Cluster, состоящий из трёх шард с тремя репликами.

### Схема:

![11-04-01](https://user-images.githubusercontent.com/1122523/114282923-9b16f900-9a4f-11eb-80aa-61ed09725760.png)

Реализуем проект создания Redis Cluster на базе облачного провайдера Yandex cloud.

Готовим нашу инфраструктуру:

Создаем рабочую директорию и копируем персональные переменные для доступа к облаку и настройки провайдера

```
mkdir redis_cluster
mkdir redis_cluster/terraform
cp terraform/04/src/personal.auto.tfvars  redis_cluster/terraform/personal.auto.tfvars
cp terraform/04/src/providers.tf  redis_cluster/terraform/providers.tf
```

Настройки провайдера providers.tf:
```
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
  required_version = ">= 0.13"
}

provider "yandex" {
  token     = var.token
  cloud_id  = var.cloud_id
  folder_id = var.folder_id
  zone      = var.default_zone
}
```

Инициализируем рабочую директорию
```
terraform init
```
![Alt_text](https://github.com/LeonidKhoroshev/micros-homeworks/blob/main/11-microservices-02-principles/screenshots/micros11.png)

В файле main.tf прописываем основные настройки кластера:
```
resource "yandex_mdb_redis_cluster" "netology" {
  name                = var.cluster_name
  environment         = var.environement
  network_id          = var.network_id
  security_group_ids  = [ "enp47qss2ianln64rt5f" ]
  sharded             = var.shard
  tls_enabled         = var.tls
  deletion_protection = var.protection

  config {
    password = var.password
    version  = var.redis_version
  }

  resources {
    resource_preset_id = var.resource_preset_id
    disk_type_id       = var.disk_type_id
    disk_size          = var.disk_size
  }
```

Значения переменных прописываем в variables.tf:
```
###cluster vars

variable "cluster_name" {
  type        = string
  default     = "netology"
}
variable "environement" {
  type        = string
  default     = "PRESTABLE"
  description = "PRODUCTION or PRESTABLE"
}
variable "network_id" {
  type        = string
  default     = "enpv78njcbpmnjc8fgr1"
}
variable "shard" {
  type        = bool
  default     = "true"
}
variable "tls" {
  type        = bool
  default     = "true"
}
variable "protection" {
  type        = bool
  default     = "false"
}
variable "password" {
  type        = string
  sensitive   = true
  default     = "....."
}
variable "redis_version" {
  type        = string
  default     = "7.0"
  description = "available for yandex_cloud now only 7.0 and 6.4"
}
variable "resource_preset_id" {
  type        = string
  default     = "b3-c1-m4"
}
variable "disk_type_id" {
  type        = string
  default     = "network-ssd"
}
variable "disk_size" {
  type        = number
  default     = "20"
```

Так как по условию задания должно быть 3 шарда, создаем 3 ВМ под каждый шард, для чего в main.tf добавляем следующий блок
```
dynamic "host" {
    for_each = { for idx, shard_name in var.shard_names : shard_name => idx }
    content {
      zone         = var.zones[(host.value + length(var.zones)) % length(var.zones)]
      subnet_id    = var.subnet_ids[(host.value + length(var.subnet_ids)) % length(var.subnet_ids)]
      shard_name   = host.key
    }
```

Под данный блок объявляем переменные

```
###host_vars

variable "shard_names" {
  description = "names of shards"
  type        = list(string)
  default     = ["shard1", "shard2", "shard3"]
}

variable "zones" {
  type        = list(string)
  default     = ["ru-central1-a", "ru-central1-b", "ru-central1-d"]
}

variable "subnet_ids" {
  type        = list(string)
  default     = ["e9b1bo21rhd37h7ajmmp", "e2lv0ngttnjk9a8htaeo", "fl8rv6gi0ngimnptkjfu"]
}
```

Далее попытался настроить репликацию следующим блоком кода в main.tf:
```
  resource "yandex_mdb_redis_replica" "netology" {
    count = length(yandex_mdb_redis_cluster.netology.host) # Количество реплик равно количеству хостов в кластере

    cluster_id     = yandex_mdb_redis_cluster.netology.id
    zone           = var.zones[(count.index + 1) % length(var.zones)] # Реплика в следующей зоне
    subnet_id      = var.subnet_ids[(count.index + 1) % length(var.subnet_ids)] # Реплика в следующей подсети
    shard_name     = "${yandex_mdb_redis_cluster.netology.host[count.index].shard_name}-replica"
    source_cluster = yandex_mdb_redis_cluster.netology.host[count.index].id
  }
```
Однако, получил следующую ошибку


![Alt_text](https://github.com/LeonidKhoroshev/micros-homeworks/blob/main/11-microservices-02-principles/screenshots/micros14.png)

Нашел следующее объяснение:
В Yandex Cloud Terraform Provider нет ресурса yandex_mdb_redis_replica. Репликация кластеров Redis в Yandex.Cloud производится автоматически, когда вы создаете кластер Redis. Поэтому,вам не нужно создавать ресурс yandex_mdb_redis_replica в явном виде. Вместо этого, у вас уже есть ресурс yandex_mdb_redis_cluster, который представляет собой кластер Redis. Репликация внутри этого кластера будет автоматически управляться платформой.

Проверим создаваемые ресурсы
```
terraform plan
```
![Alt_text](https://github.com/LeonidKhoroshev/micros-homeworks/blob/main/11-microservices-02-principles/screenshots/micros15.png)
![Alt_text](https://github.com/LeonidKhoroshev/micros-homeworks/blob/main/11-microservices-02-principles/screenshots/micros16.png)

Создаем кластер
```
terraform apply
```
![Alt_text](https://github.com/LeonidKhoroshev/micros-homeworks/blob/main/11-microservices-02-principles/screenshots/micros12.png)
![Alt_text](https://github.com/LeonidKhoroshev/micros-homeworks/blob/main/11-microservices-02-principles/screenshots/micros13.png)

В качестве ответа прилагаю репозиторий [redis_cluster](https://github.com/LeonidKhoroshev/redis_cluster/tree/master)

