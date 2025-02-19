# Pod, ReplicaSet, Deployment

## Pod:

Um Pod no Kubernetes é a menor unidade implantável, que pode conter um ou mais contêineres, compartilhando recursos como rede e armazenamento. Ele é utilizado para executar aplicações em contêineres, garantindo que eles funcionem juntos de forma coesa.
[https://kubernetes.io/docs/concepts/workloads/pods/]

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80

```
#### kubectl apply/replace
Acima um arquivo yaml exemplificando a criação de um container que irá executar uma versão específica do nginx.
Para de fato criar o pod utilizando o arquivo, podemos executar o comando `kubectl apply -f nome-do-arquivo.yaml`

Conseguimos alterar algumas informações no arquivo yaml como por exemplo a imagem que está sendo utilizada no container, podemos por exemplo alterar a imagem do nginx para httpd e utilizar o comando replace para aplicar a alteração:

`kubectl replace -f pod.yaml --force --grace-period=0`

Explicando o comando acima: utilizamos o kubectl replace para aplicar a alteração feita, combinamos com o parametro force para forçar a alteração mesmo com o pod em execução e complementamos com o grace period 0, basicamente informando ao kubernetes para executar o comando de forma mais rápida sem ser pela padrão (informamos 0 segundos para conclusão para que seja alterado de forma imediata)

#### kubectl run/dry-run

Uma outra maneira, que até mesmo se torna mais eficiente para se criar pods, seria utilizar o comando --dry-run=client
O comand dry-run 'executa' uma simulação do comando, podemos passar alguns comandos do kubernetes seguidos do dry-run que de fato eles não serão executados, apenas será gerado um output no terminal

Dessa forma, com algumas combinações, conseguimos gerar um arquivo yaml de qualquer pod e imagem que desejamos utilizar.

Exemplo de utilização do comando dry-run gerando um arquivo yaml com uma estrutura pronta para se criar um pod do httpd

`kubectl run mypod --image http --dry-run=client -o yaml > pod-httpd.yaml`

No comando anterior, usamos o **run** para criação dos pods como imagem, usamos o **--dry-run=client** combinando com o parametro de **-o yaml** para gerar uma estrutura de manifesto e **direcionamos a saída do comando** para o arquivo **pod-httpd.yaml**

O resultado foi:

```yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: meupod
  name: meupod
spec:
  containers:
  - image: httpd
    name: meupod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```

