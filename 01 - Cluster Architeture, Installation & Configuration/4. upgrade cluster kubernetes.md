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

Desconecte da máquina worker-node e retorne na vm control-plane para retirarmos o modo de manutenção que foi setado com o comando `kubectl drain`.
Para isso, execute o comando `kubectl uncordon nome-do-node`
Após finalizar a execução do comando, você pode conferir se a label `SchedullingDisable` foi removida com o comando `kubectl get node`

Para finalizar, crie um pod de teste para garantir que ele será executado após as atualizações realizadas:
`kubectl run nginx --image nginx`

