# ** HW-19 - KUBERNETES-1 **

Развернул мастер и воркер по инструкциям (сначала пытался на 18.04 установить, но были ошибки, на 20.04 все ок):
1. Обновляем систему, импортируем ключи, ставим пакеты
```
# ALL
apt update && apt upgrade -y && apt install mc htop wget curl net-tools apt-transport-https ca-certificates gnupg lsb-release
# ALL - DOCKER INSTALL
mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
apt update
apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin
# ALL - KUBERNETES INSTALL
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt install kubelet kubeadm kubectl
```
2. Смотрим внешний адрес master и проводим инициализацию
```
sudo kubeadm init --apiserver-cert-extra-sans=51.250.76.199 --apiserver-advertise-address=0.0.0.0 --control-plane-endpoint=51.250.76.199 --pod-network-cidr=10.244.0.0/16
```
- При инцициализации возникла сошибка:
```
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR CRI]: container runtime is not running: output: E0701 07:16:22.125652    4047 remote_runtime.go:925] "Status from runtime service failed" err="rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.RuntimeService"
time="2022-07-01T07:16:22Z" level=fatal msg="getting status of runtime: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.RuntimeService"
, error: exit status 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```
- Решилось правкой конфига `/etc/containerd/config.toml` - нужно закоментировать строку `disabled_plugins = ["cri"]` и перезапустить службу `containerd`.
- Проводим действия далее по инструкции:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
- Запоминаем строку, которую будем использовать для подключения воркера
```
kubeadm join 51.250.76.199:6443 --token k7n61t.fz1bbfvf7mmqcylo --discovery-token-ca-cert-hash sha256:bba278433084cea8f14af7e838925345eff47d9234e7a6af71779e575846a5d0
```
3. На воркере выполняем (выполнилось без ошибок только после ребута сервера и правки конфига containerd):
```
kubeadm join 51.250.76.199:6443 --token k7n61t.fz1bbfvf7mmqcylo --discovery-token-ca-cert-hash sha256:bba278433084cea8f14af7e838925345eff47d9234e7a6af71779e575846a5d0
```
Получаем:
```
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```
- Проверяем ноды на мастере и получаем статус NotReady:
```
yc-user@master:~$ kubectl get nodes
NAME     STATUS     ROLES           AGE    VERSION
master   NotReady   control-plane   14m    v1.24.2
worker   NotReady   <none>          5m2s   v1.24.2
```
3. Ставим сетевой плагин:
```
curl https://docs.projectcalico.org/v3.20/manifests/calico.yaml -o calico.yaml
sudo nano calico.yaml
# Раскомментируем (важно соблюсти отступы, иначе будет ошибка) 
- name: CALICO_IPV4POOL_CIDR
  value: "10.244.0.0/16"
# Применяем
kubectl apply -f calico.yaml
```
- Снова проверяем ноды - все ок!
```
yc-user@master:~$ kubectl get nodes
NAME     STATUS   ROLES           AGE   VERSION
master   Ready    control-plane   24m   v1.24.2
worker   Ready    <none>          14m   v1.24.2
yc-user@master:~$
```
