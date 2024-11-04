# Sprint 3
## Пояснения перед работой, лично мое мнение, могу ошибаться. 

** Пояснение 1 **

Задание немного противоречит само себе в части устройств.
Пользователь самостоятельно выбирает необходимые ему модули умного дома. Компания но поддерживает подключение к экосистеме устройств партнёров.

Но при этом. Модули управления приборами и сами приборы (устройства) должны быть максимально готовы к использованию и продаваться в отдельных комплектах для удобной покупки и подключения.

Т.е. это либо стандартизированные наборы (расширяемые количественно однотипными моделями), например комплекты безопасность, отопление, сантехника (протечки и подача воды), и т.п.
Или наборы из датчиков и контроллеров, которые могут объединяться в любые комплекты с помощью настроек (например наборы tuya ).

** Пояснение 2. **

Если рассматривать задачу, только как работу с готовыми комплектами, тогда данную задачу можно попробовать решить с помощью модульного подхода к программированию. Каждый модуль будет равен комплекту (например модуль безопасность – комплект безопасность).
Однако при неконтролируемом расширении (в задании указано, что пользователь сам может расширять комплекты. Ограничения не указаны) модули скорее всего получатся достаточно тяжелыми. Это частично решит поставленную задачу. Поэтому от данного решения нужно отказаться.

** Отмазки **

Я не рассматриваю ситуацию с временным сохранением монолита в to be. Т.е. as-is монолит, to-be транзитная монолит + микросервисы, to-be целевая только микросервисы. Т.к. монолит в себе содержит условно только два домена, мониторинг температуры(можно было бы временно использовать), управление устройствами, и вот повесить все новые устройства на эту часть монолита без возможности масштабирования. В моем понимание, самоубийство.  А сохранение монолита только для управления температурой двойная работа.

Я не рассматриваю какую –то офлайн жизнь комплектов. Понятно, что она в реальной жизни должна быть. Но по условиям, должен быть канал для работы, и считаем, условно – он идеавлен, и никогда не падает. 

# Задание 1. Анализ и планирование
## 1.3 Определите домены и границы контекстов. 

**Домены и границы контекстов:**

На основе монолита можно выделить следующие домены:

1. **Управление Устройствами (Device Management)**:
   - Включение/выключение отопления.
   - Установка целевой температуры.

2. **Мониторинг Температуры (Temperature Monitoring)**:
   - Получение текущей температуры.

# Базовая настройка

## Запуск minikube

[Инструкция по установке](https://minikube.sigs.k8s.io/docs/start/)

```bash
minikube start
```

## Добавление токена авторизации GitHub

[Получение токена](https://github.com/settings/tokens/new)

```bash
kubectl create secret docker-registry ghcr --docker-server=https://ghcr.io --docker-username=<github_username> --docker-password=<github_token> -n default
```

## Установка API GW kusk

[Install Kusk CLI](https://docs.kusk.io/getting-started/install-kusk-cli)

```bash
kusk cluster install
```

## Смена адреса образа в helm chart

После того как вы сделали форк репозитория и у вас в репозитории отработал GitHub Action. Вам нужно получить адрес образа <https://github.com/><github_username>/architecture-sprint-3/pkgs/container/architecture-sprint-3

Он выглядит таким образом
```ghcr.io/<github_username>/architecture-sprint-3:latest```

Замените адрес образа в файле `helm/smart-home-monolith/values.yaml` на полученный файл:

```yaml
image:
  repository: ghcr.io/<github_username>/architecture-sprint-3
  tag: latest
```

## Настройка terraform

[Установите Terraform](https://yandex.cloud/ru/docs/tutorials/infrastructure-management/terraform-quickstart#install-terraform)

Создайте файл ~/.terraformrc

```hcl
provider_installation {
  network_mirror {
    url = "https://terraform-mirror.yandexcloud.net/"
    include = ["registry.terraform.io/*/*"]
  }
  direct {
    exclude = ["registry.terraform.io/*/*"]
  }
}
```

## Применяем terraform конфигурацию

```bash
cd terraform
terraform init
terraform apply
```

## Настройка API GW

```bash
kusk deploy -i api.yaml
```

## Проверяем работоспособность

```bash
kubectl port-forward svc/kusk-gateway-envoy-fleet -n kusk-system 8080:80
curl localhost:8080/hello
```

## Delete minikube

```bash
minikube delete
```
