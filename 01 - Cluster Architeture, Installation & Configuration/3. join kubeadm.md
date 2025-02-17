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
