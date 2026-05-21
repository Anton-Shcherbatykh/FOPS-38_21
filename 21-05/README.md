## Домашнее задание к занятию «Хранение в K8s» FOPS-38 (Щербатых А.Е.)

---

**Цель задания**

Научиться работать с хранилищами в тестовой среде Kubernetes:

- обеспечить обмен файлами между контейнерами пода;
- создавать PersistentVolume (PV) и использовать его в подах через PersistentVolumeClaim (PVC);
- объявлять свой StorageClass (SC) и монтировать его в под через PVC.

Для этих целей с помощью Terraform развернул ВМ в Yandex.Cloud

![alt text](Pictures/pic01.jpg)

и установил на ней MikroK8s (работал с ним при выполнении предыдущих заданий, показался удобным).

![alt text](Pictures/pic02.jpg)

---

### Задание 1. Volume: обмен данными между контейнерами в поде

### Задача

Создать Deployment приложения, состоящего из двух контейнеров, обменивающихся данными.

Шаги выполнения

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.
2. Настроить busybox на запись данных каждые 5 секунд в некий файл в общей директории.
3. Обеспечить возможность чтения файла контейнером multitool.

### Ответ 1

Создаю манифест ```containers-data-exchange.yaml```. Применяю его командой ```microk8s kubectl apply -f containers-data-exchange.yaml``` и проверяю, что Pod запустился командой ```microk8s kubectl get pods -l app=data-exchange```.

![alt text](Pictures/pic03.jpg)

Затем получаю описание Pod'а командой ```microk8s kubectl describe pod -l app=data-exchange```

![alt text](Pictures/pic04.jpg)

![alt text](Pictures/pic04_1.jpg)

После описанных выше манипуляций проверяю чтение файла контейнером ```multitool```

![alt text](Pictures/pic05.jpg)

Командой ```microk8s kubectl exec -it deployment/data-exchange -c multitool-reader -- tail -f /data/shared.log``` для чтения файла в реальном времени подключаюсь к контейнеру ```multitool-reader``` и запускаю в нём ```tail -f```

![alt text](Pictures/pic05_1.jpg)

