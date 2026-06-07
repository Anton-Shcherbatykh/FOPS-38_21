## Домашнее задание к занятию «Установка Kubernetes» FOPS-38 (Щербатых А.Е.)

### Задание 1. Установить кластер k8s с 1 master node

1. Подготовка работы кластера из 5 нод: 1 мастер и 4 рабочие ноды.
2. В качестве CRI — containerd.
3. Запуск etcd производить на мастере.
4. Способ установки выбрать самостоятельно.

---

### Ответ 1.

Для выполнения задания развернул 5 5 виртуальных машин (ВМ) с Ubuntu 20.04 LTS в Yandex.Cloud, которые имеют сетевой доступ друг к другу.

![alt text](Pictures/pic00.jpg)

![alt text](Pictures/pic01.jpg)

Затем настроим все ВМ.

```bash
# 1.1 Отключаем Swap (В официальной документации Kubernetes и лучших практиках сообщества настоятельно рекомендуется отключать swap на нодах, где планируется запускать рабочую нагрузку Kubernetes. Это считается стандартной практикой для поддержания стабильности и производительности кластера.)
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

# 1.2 Прописываю загрузку модулей ядра для работы сетевых мостов (overlay, br_netfilter)
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

# 1.3 Настраиваю параметры сети (iptables видит трафик мостов, включение IP-форвардинга)
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

# 1.4 Прменяю настройки sysctl
sudo sysctl --system

# 1.5 Устанавливаю необходимые зависимости
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl

# 1.6 Устанавливаю containerd в качестве CRI (это позволит обеспечить взаимодействие между компонентами кластера и средой выполнения контейнеров).
sudo apt-get install -y containerd
```

![alt text](Pictures/pic02.jpg)

Затем создаю конфигурацию для containerd и перезапускаю его.

```bash
# 2.1 Генерация конфигурационного файла по умолчанию
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# 2.2 Настройка systemd в качестве cgroup driver ( для корректного управления ресурсами и предотвращения конфликтов в системе)
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

# 2.3 Перезапуск containerd
sudo systemctl restart containerd
sudo systemctl enable containerd
```

Устанавливаю компоненты Kubernetes ```kubelet```, ```kubeadm``` и ```kubectl```.

Для этого сначала создаю директорию для ключей и устанавливаю права доступа:

```bash
sudo mkdir -p -m 755 /etc/apt/keyrings
```

Затем 

```bash
# 3.1 Добавляю официальный репозиторий Kubernetes
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# 3.2 Устанавливаю компоненты
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

# 3.3 Блокирую версии для предотвращения автоматического обновления (В Kubernetes существует политика допустимого расхождения версий между некоторыми компонентами (например, между ```kube-apiserver```, ```kubelet``` и ```kubectl```). Блокировка версий помогает поддерживать согласованность между компонентами, что снижает риск конфликтов при обновлении).
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```

![alt text](Pictures/pic03.jpg)
