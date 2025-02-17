### **Configuração worker-node**

Iremos repetir basicamente o mesmo processo anterior, agora para a máquina worker-node.
Utilize a mesma chave ssh criada anteriormente e realize uma conexão na máquina para execução dos comandos abaixo:

1. INSERINDO OS MODULOS DE KERNEL

`cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF`

2. carregando os módulos

`sudo modprobe overlay`
`sudo modprobe br_netfilter`

3.

`cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.ipv4.ip_forward = 1
net.ipv4.conf.all.forwarding = 1
net.ipv6.conf.all.forwarding = 1
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.conf.all.rp_filter = 0
net.ipv6.conf.all.rp_filter = 0
EOF`

4. `sudo sysctl --system`

**Containerd**

`apt install -y containerd`

`mkdir -p /etc/containerd`

`containerd config default>/etc/containerd/config.toml`

`sed -i 's/SystemdCgroup.*/SystemdCgroup = true/g' /etc/containerd/config.toml`

`sudo systemctl enable --now containerd`
`sudo systemctl status containerd`

**Instalação dos componentes kubectl kubeadm kubelet**

Documentação oficial:
[https://v1-31.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/]

1. 
- `sudo apt-get update`
- `sudo apt-get install -y apt-transport-https ca-certificates curl gpg`
    
2. 
- `curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg`

3. 
- `echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list`

4. 
- `sudo apt-get update`
- `sudo apt-get install -y kubelet kubeadm kubectl`
- `sudo apt-mark hold kubelet kubeadm kubectl` #comando utilizado para marcar os componentes kubeadm,kubelet e kubectl de modo que eles não sofram atualizações.

5.
- `sudo systemctl enable --now kubelet`

Verifique se o kubelet foi inicializado corretamente com o comando `sudo systemctl status kubelet`
Verifique se o containerd foi inicializado corretamente com o comando `sudo systemctl status containerd`

Execute o comando `reboot` para reiniciar a vm worker-node e retorne na vm control-plane.
