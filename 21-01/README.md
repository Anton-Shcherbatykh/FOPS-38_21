## Домашнее задание к занятию «Kubernetes. Причины появления. Команда kubectl» FOPS-38 (Щербатых А.Е)

### Цель задания

Для экспериментов и валидации ваших решений вам нужно подготовить тестовую среду для работы с Kubernetes. Оптимальное решение — развернуть на рабочей машине или на отдельной виртуальной машине MicroK8S.

**Чеклист готовности к домашнему заданию**

Личный компьютер с ОС Linux или MacOS

или

ВМ c ОС Linux в облаке либо ВМ на локальной машине для установки MicroK8S

**Инструкция к заданию**

Установка MicroK8S:

- sudo apt update,
- sudo apt install snapd,
- sudo snap install microk8s --classic,
- добавить локального пользователя в группу sudo usermod -a -G microk8s $USER,
- изменить права на папку с конфигурацией sudo chown -f -R $USER ~/.kube.

Полезные команды:

- проверить статус microk8s status --wait-ready;
- подключиться к microK8s и получить информацию можно через команду microk8s command, например, microk8s kubectl get nodes;
- включить addon можно через команду microk8s enable;
- список addon microk8s status;
- вывод конфигурации microk8s config;
- проброс порта для подключения локально microk8s kubectl port-forward -n kube-system service/kubernetes-dashboard 10443:443.

Настройка внешнего подключения:

- отредактировать файл /var/snap/microk8s/current/certs/csr.conf.template

```bash
# [ alt_names ]
# Add
# IP.4 = 123.45.67.89
```

- обновить сертификаты ```sudo microk8s refresh-certs --cert front-proxy-client.crt```.

Установка kubectl:

```curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl;
chmod +x ./kubectl;
sudo mv ./kubectl /usr/local/bin/kubectl```

настройка автодополнения в текущую сессию ```bash source <(kubectl completion bash)```;
добавление автодополнения в командную оболочку ```bash echo "source <(kubectl completion bash)" >> ~/.bashrc```.
Инструменты и дополнительные материалы, которые пригодятся для выполнения задания
Инструкция по установке MicroK8S.
Инструкция по установке автодополнения kubectl.
Шпаргалка по kubectl.
Задание 1. Установка MicroK8S
Установить MicroK8S на локальную машину или на удалённую виртуальную машину.
Установить dashboard.
Сгенерировать сертификат для подключения к внешнему ip-адресу.
Задание 2. Установка и настройка локального kubectl
Установить на локальную машину kubectl.
Настроить локально подключение к кластеру.
Подключиться к дашборду с помощью port-forward.

