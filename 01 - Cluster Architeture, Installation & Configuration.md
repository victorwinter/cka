# **Cluster Architeture, Installation & Configuration**

Para a prática do cenário de Cluster Architecture, Installation & Configuration da certificação Certified Kubernetes Administrator (CKA), este repositório utilizará duas máquinas virtuais (VMs) provisionadas na AWS (Amazon Web Services), com a configuração t2.medium. 
Esse ambiente simula um cenário de cluster Kubernetes com dois nós, um provisionado para configurações e elementos do control-plane e outro relacionado ao worker node
Será utilizado o kubeadm para configurar e gerenciar o cluster Kubernetes. Isso é feito porque o kubeadm é uma das ferramentas e abordagens oficialmente cobradas no exame CKA.

Esse cenário é parte do exame CKA descrito no curriculum definido pela Cloud Native Compunting Foundation (CNCF)
[https://github.com/cncf/curriculum/blob/master/CKA_Curriculum_v1.31.pdf]

## **Detalhes da configuração:**
- Plataforma: AWS (Amazon Web Services).
- Tipo de Instância: 2 máquinas virtuais com o tipo de instância t2.medium 2vCpu e 4Gb de Memória.
- Sistema operacional: Ubuntu Server 22.04 LTS 64 Bit.
- Uso: Uma máquina virtual será configurada como Master Node, e a outra como Worker Node.
- Crie um novo Security Group (SG) para as instâncias do Kubernetes, garantindo que ele permita o acesso SSH (porta 22) de sua máquina local (ou de um intervalo de IPs seguro).
- Storage 30GiB
- Crie uma chave SSH para posterior conexão utilizando sua máquina local com um terminal de sua preferência
- Durante a criação das máquinas EC2, copie o id gerado pelo security group e acesse a seção de Edit Inbound Rules
- Clique em Add Rule, em type selecione 'All traffic' e em source selecione o id do próprio security group que havia copiado.
Obs.: Ressalta-se que essa não é a melhor opção, porém para exemplificação do cenário da prova essa liberação será necessária, essa liberação não deve ser utilizada em um ambiente de produção.
- Renomeie as máquinas de forma que uma seja seu control-plane e outra o worker-node, nesse cenário as máquinas terão os nomes cka-control-plane e cka-node
- 
### **Configuração node Control Plane**
  
**Conectando nas máquinas EC2:**
- Após realizar o download da chave (.pem) gerada pela console aws 
- Copie a chave para um diretório de sua preferência
- Conceda a permissao chmod 400 para a chave que está no diretório.
- Realize uma conexão ssh na máquina definida como control-plane
  `ssh ubuntu@ip-do-host -i chave.pem`

Para esse primeiro cenário, iremos seguir a documentação oficial do Kubernetes adicionando a versão v1.31 para posteriormente executarmos a atualização.
[https://v1-31.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/]

Conectado na máquina EC2 control plane, execute os comandos abaixos conforme descrito na documentação oficial para instalação dos componentes:

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

**Containerd:**
Para a configuração do Kubernetes neste cenário, o containerd será utilizado como o runtime de contêineres. 
O containerd é uma parte essencial do ecossistema Kubernetes, responsável pela execução e gerenciamento de contêineres.
Embora o containerd seja um componente fundamental para o funcionamento do Kubernetes, é importante ressaltar que a configuração do containerd não será cobrada no exame CKA, pois ele já virá pré-configurado em muitas distribuições e ambientes Kubernetes.
O objetivo de demonstrar a configuração além de melhor entendimento e aprofundamento no cenário é mostrar como é executada.
Abaixo a sequência de comandos que deverão ser utilizados:

1 - Ativar dois módulos de kernel: Overlay e br_netfilter

`cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF`

2. Carregue os dois módulos
- `sudo modprobe overlay`
- `sudo modprobe br_netfilter`

3. Habilitando algumas flags, ex: ipforward, netbridge
Crie um arquivo no caminho:

`/etc/sysctl.d/kubernetes.conf`

Edite o arquivo e insira o conteúdo:

`net.ipv4.ip_forward = 1
net.ipv4.conf.all.forwarding = 1
net.ipv6.conf.all.forwarding = 1
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.conf.all.rp_filter = 0
net.ipv6.conf.all.rp_filter = 0`

5. Após inserir o conteúdo e salvar o arquivo, execute o comando para carregar os módulos/flags:
- `sudo sysctl --system`

5. Instalação containerd
- `apt install -y containerd`
- `mkdir -p /etc/containerd`

6. Gere o arquivo de configuração padrão do containerd e redirecione a saída para um arquivo config.toml
- `containerd config default>/etc/containerd/config.toml`

7. Altere o valor do conteúdo config.toml modificando o valor SystemdCgroup para true, para isso, pode-se utilizar o comando abaixo para substituir todos os valores:
- `sed -i 's/SystemdCgroup.*/SystemdCgroup = true/g' /etc/containerd/config.toml`

8. Inicie o containerd
- `systemctl enable containerd`
- `systemctl start containerd`
- `systemctl status containerd`

Pré-requisitos instalados.

**Kubeadm Init:**
[https://v1-31.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/]

1. Após executar todos os passos anteriores e certificar de que os serviços estão instalados e em execução conforme esperado.
Iremos inicializar o master node de um cluster Kubernetes, configurando o plano de controle, criando os certificados necessários e gerando o comando para adicionar nós de trabalho (worker nodes) ao cluster, utilize o comando:
- `kubeadm init`

2. Finalizando a criação do cluster com o comando kubeadm init, execute os comandos solicitados na finalização do processo para a conclusão:
- `mkdir -p $HOME/.kube`
- `sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config`
- `sudo chown $(id -u):$(id -g) $HOME/.kube/config`

**Container Network Interface (CNI)**

A instalação da CNI em si, não é cobrada na prova de ceritificação, se houver algo relacionado, todos os comandos necessários e os steps serão informados durante a prova.
Para que nosso node criado nos passos anteriores fique com status de Ready, será necessário realizar a configuração da CNI

Iremos utilizar a documentação da Cilium [https://docs.cilium.io/en/v1.14/installation/k8s-install-kubeadm/#installation-using-kubeadm]

1. Copie e execute o comando para baixar e instalar o binário da Cilium:
`CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}`

2. Execute a instalação
- `cilium install`

3. Verifique o status da instalação
- `cilium status`

4. Verifique se o node está disponível (Ready)
- `kubectl get node`

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

### Executando o JOIN no cluster

Para que a vm control-plane tenha visibilidade do cluster worker-node e para que sejam 'integrados' corretamente, é necessário executarmos o comando `kubeadm join`,  é o processo de adicionar um novo nó (worker ou master) a um cluster Kubernetes existente.

Conectado na VM control-plane, iremos executar um comando que irá retornar um token temporário para autenticação de um nó ao se juntar a um cluster Kubernetes e imprime o comando completo que deve ser executado no nó que está sendo adicionado (worker node).

Para obter o comando completo, na VM control-plane execute:
`kubeadm token create --print-join-command`

O retorno do comando anterior, será algo semelhante as informações abaixo:

`kubeadm join 192.168.1.100:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:bc4ac34b9d0f8d2f8c1c0b7fbe59b072d4fe09bdb16c5f9b67d8fd95fbaea4d0`

Copie o retorno do comando completo e volte na VM worker-node, após conectar na VM usando o usuário root cole o comando completo copiado anteriormente.

Executando o comando corretamente, o join será executado e a confirmação de sucesso será algo semelhante:

`Joining the cluster using kubeadm join, please wait...
This node has joined the cluster:`
`The cluster control plane is now aware of this node, and this node is part of the cluster.`
`The kubelet has been started on this node.`
`The node has been successfully registered as a worker.`

`Run 'kubectl get nodes' on the master node to see this node in the cluster.`

Para conferir se o processo foi executado corretamente, execute o comando `kubectl get nodes` e verifique se o nó worker é visível.

## Realizando o upgrade no cluster

Como um dos tópicos cobrados nos exames CNCF, é a atualização dos componentes kubelet, kubectl e kubeadm além da preparação necessária no node para realização do upgrade.

Neste cenário, utilizamos a versão 1.31 dos componentes e agora iremos realizar o upgrade para a versão 1.32
Para isso, utilizaremos basicamente os mesmos comandos no início durante a instalação, porém alterando agora para a versão desejada (1.32)
[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/]

1. `curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg`

2. `echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list`

3. `sudo apt update`

4. `apt-mark unhold kubeadm kubelet kubectl` #comando para tirar a trava que impedia a atualização dos componentes

5. `apt install kubeadm kubelet kubectl`

6.` kubeadm version` #comando para conferir se de fato foi atualizada a versão para 1.32

7. `kubeadm upgrade plan`

O comando `kubeadm upgrade plan` exibe um plano detalhado sobre as atualizações disponíveis para o cluster Kubernetes. Ele verifica a versão atual do Kubernetes e informa quais atualizações de componentes (como o control plane, kubelet e kubectl) estão disponíveis, além de apresentar recomendações sobre a versão que pode ser aplicada.

Após executar este comando, ele irá retornar o próximo comando que deve ser executado para aplicar a atualização, algo semelhante a: `kubeadm upgrade apply v1.32.1`

Utilize o comando `kubeadm upgrade status` para verificar o andamento.

Feito todos os passos anteriores, como boa prática vamos marcar os pacotes novamente para não sofrerem atualizações:

`apt-mark hold kubelet kubeadm kubectl`

### Preparando o node control-plane (drain)

O comando `kubectl drain` no Kubernetes é usado para preparar um nó para manutenção, desalocando todos os pods de um nó específico de forma segura.

Conectado a vm control-plane iremos executar o comando `kubectl get node` para verificar qual o nome do worker-node
Após copiar o nome retornado do nó, execute o comando de exemplo abaixo, no lugar de `ip-172-31-2-48` informe o respectivo nome do nó worker.
`kubectl drain ip-172-31-2-48 --ignore-daemonsets --force`

Após a finalização do comando, desconecte da VM control-plane e retorne a conexão na VM worker-node

### Atualização dos componentes na vm worker-node

Realizar o procedimento de update dos componentes kubeadm, kubelet e kubectl na vm workernode
- `curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg`

- `echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list`

- `sudo apt update`

- `apt-mark unhold kubeadm kubelet kubectl` 

- `apt install kubeadm kubelet kubectl`

- `kubeadm version` #comando para conferir se de fato foi atualizada a versão para 1.32

- `kubeadm upgrade node`

- `apt-mark hold kubelet kubectl kubeadm`

- Atualização finalizada


