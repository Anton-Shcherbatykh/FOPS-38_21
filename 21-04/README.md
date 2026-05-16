## Домашнее задание к занятию «Сетевое взаимодействие в Kubernetes» FOPS-38 (Щербатых А.Е.)

---

### Задание 1: Настройка Service (ClusterIP и NodePort)

**Задача**

Развернуть приложение из двух контейнеров (nginx и multitool) и обеспечить доступ к ним:

- Внутри кластера через ClusterIP.
- Снаружи через NodePort.

**Шаги выполнения**

1. Создать Deployment с двумя контейнерами:

- nginx (порт 80).
- multitool (порт 8080).
- Количество реплик: 3.

2. Создать Service типа ClusterIP, который:

- Открывает nginx на порту 9001.
- Открывает multitool на порту 9002.

3. Проверить доступность изнутри кластера:

```bash
 kubectl run test-pod --image=wbitt/network-multitool --rm -it -- sh
 curl <service-name>:9001 # Проверить nginx
 curl <service-name>:9002 # Проверить multitool
```

4. Создать Service типа NodePort для доступа к nginx снаружи.
5. Проверить доступ с локального компьютера:

```curl <node-ip>:<node-port>```

или через браузер.

---

## Задание 2: Настройка Ingress

**Задача**

Развернуть два приложения (frontend и backend) и обеспечить доступ к ним через Ingress по разным путям.

**Шаги выполнения**

1. Развернуть два Deployment:

- frontend (образ nginx).
- backend (образ wbitt/network-multitool).

2. Создать Service для каждого приложения.
3. Включить Ingress-контроллер:

```microk8s enable ingress```

4. Создать Ingress, который:

- Открывает frontend по пути /.
- Открывает backend по пути /api.

5. Проверить доступность:

 ```bash
 curl <host>/
 curl <host>/api
```

или через браузер.

---

### Ответ 1.

1. Создайте манифест Deployment с двумя контейнерами и 3 репликами
Файл: deployment-multi-container.yaml

 ```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool:latest
        env:
        - name: HTTP_PORT
          value: "8080"
        ports:
        - containerPort: 8080
```

Пояснение: multitool по умолчанию слушает порт 80, но переменная HTTP_PORT=8080 заставляет его использовать порт 8080. Это исключит конфликт с nginx.

Примените:

```bash
microk8s kubectl apply -f deployment-multi-container.yaml
```

Проверьте, что все три реплики запустились:

```bash
microk8s kubectl get pods -l app=web-app
```

Каждый под должен иметь статус Running и 2/2 READY.

2. Создайте Service типа ClusterIP (два порта)
Файл: service-clusterip.yaml

```bash
apiVersion: v1
kind: Service
metadata:
  name: web-svc-clusterip
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
  - name: nginx-port
    protocol: TCP
    port: 9001
    targetPort: 80
  - name: multitool-port
    protocol: TCP
    port: 9002
    targetPort: 8080
```

Примените:

```bash
microk8s kubectl apply -f service-clusterip.yaml
```

Посмотрите, какой IP получил сервис:

```bash
microk8s kubectl get svc web-svc-clusterip
```

3. Проверка доступа изнутри кластера (через тестовый под)
4. 
Создайте временный под с multitool и выполните curl:

```bash
microk8s kubectl run test-pod --image=wbitt/network-multitool --rm -it --restart=Never -- sh
```

Когда окажетесь внутри контейнера, выполните:

```bash
curl http://web-svc-clusterip:9001   # Должна появиться страница nginx
curl http://web-svc-clusterip:9002   # Должна появиться информация от multitool
```

Выйдите из пода, набрав exit или нажав Ctrl+D.

Под автоматически удалится.

Скриншот вывода curl приложите к отчёту.

4. Создайте Service типа NodePort для доступа к nginx снаружи
Файл: service-nodeport.yaml

```bash
apiVersion: v1
kind: Service
metadata:
  name: web-svc-nodeport
spec:
  type: NodePort
  selector:
    app: web-app
  ports:
  - name: nginx-port
    protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080   # можно указать явно, либо Kubernetes назначит в диапазоне 30000-32767
```

Примените:

```bash
microk8s kubectl apply -f service-nodeport.yaml
```

Узнайте назначенный порт (если не указали 30080):

```bash
microk8s kubectl get svc web-svc-nodeport
```

В колонке PORT(S) будет что-то вроде 80:30080/TCP.

5. Проверка доступа с локального компьютера (хоста Windows)

Вам понадобится IP-адрес виртуальной машины, который доступен с вашего Windows.

Если вы используете режим NAT в VirtualBox, нужно настроить проброс порта (порт хоста, например 30080 → порт гостя 30080) или переключить сеть на мост (Bridge). Проще всего — использовать мост, чтобы ВМ получила IP вашей локальной сети.

Узнайте IP ВМ: в Debian выполните ```ip a | grep "inet " | grep -v 127.0.0.1. Если у вас мост, вы увидите адрес типа 192.168.1.x```.

На Windows откройте командную строку и выполните:

```bash
curl http://<IP_ВМ>:30080
```
Или откройте браузер и перейдите по адресу ```http://<IP_ВМ>:30080```.

Вы должны увидеть страницу ```Welcome to nginx!```
