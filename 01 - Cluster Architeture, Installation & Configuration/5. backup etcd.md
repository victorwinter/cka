## Backup etcd

Requisitos:

- realizar a instalação do etcdctl [apt install etcd-client] (na prova esse componente já estará instalado)

Hands on:

- após instalar o etcdctl no cluster control-plane, iremos executar o comando abaixo:

ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> snapshot save <backup-file-location>

Para preenchimento dos parâmetros, como cacert, cert, key, precisaremos do caminho onde estão esses arquivos e, essas informações estão disponíveis no arquivo yaml do diretório /etc/kubernetes/manifests/etcd.yaml

- trusted-ca-file
- cert-file
- key-file

Comando com os parâmetros necessários preenchidos:

ETCDCTL_API=3 etcdctl snapshot save /tmp/snapshot-cka.db   --endpoints=https://127.0.0.1:2379   --cacert=/etc/kubernetes/pki/etcd/ca.crt   --cert=/etc/kubernetes/pki/etcd/server.crt   --key=/etc/kubernetes/pki/etcd/server.key

Obtendo informações sobre o arquivo de backup gerado:

ETCDCTL_API=3 etcdctl snapshot status /tmp/snapshot-cka.db --write-out=table

RESTORE:

Para restaurar o backup em um cluster, utilizaremos basicamente o mesmo comando anterior, alterando a ação para restore e adicionando mais parâmetros como o --data-dir=DIRETORIO-PARA-DESCOMPACTAR-O-BACKUP

ETCDCTL_API=3 etcdctl snapshot restore /tmp/snapshot-cka.db --data-dir=/var/lib/etcd-backup --endpoints=https://127.0.0.1:2379   --cacert=/etc/kubernetes/pki/etcd/ca.crt   --cert=/etc/kubernetes/pki/etcd/server.crt   --key=/etc/kubernetes/pki/etcd/server.key

Após realizar o restore e salvar o arquivo informado no diretório /var/lib, deveremos acessar os statics pods do cluster que ficam em /etc/kubernetes/manifests editar o arquivo YAML etcd.yaml e alterar o caminho do path de montagem em 3 pontos:

em volumeMounts > mounthPath
hostPath > path e data_dir

Onde estiver o caminho /var/lib/etcd , deverá ser alterado para o caminho onde está o arquivo de backup restaurado anteriormente, no nosso caso ficou no mesmo diretório /var/lib/etcd-backup
