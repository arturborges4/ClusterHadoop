# ClusterHadoop
Cluster Hadoop utilizando VMs Ubuntu Server no Proxmox


🖥️Configuração do Cluster
<h1>1. Função de cada nó no cluster Hadoop:</h1>

<h3>NameNode (Master):</h3>

É responsável por gerenciar o metadata do sistema de arquivos distribuído HDFS (ex.: nomes dos arquivos, blocos, localizações nos DataNodes).
Coordena as operações de leitura/escrita e é essencial para a operação do cluster.

<h3>DataNode (Slaves):</h3>

Armazenam os dados reais em blocos.
Respondem às solicitações do NameNode para leitura, escrita e replicação de dados.

<h3>ResourceManager (Master):</h3>

Gerencia os recursos do cluster para execução de aplicações distribuídas.
Trabalha com o NodeManager para alocar contêineres e monitorar tarefas.

<h3>NodeManager (Slaves):</h3>

Executa tarefas atribuídas pelo ResourceManager.
Monitora o uso de recursos (CPU, memória) em cada nó.

<h2>2. Configurações alteradas nos arquivos do Hadoop e motivos:</h2>

<h3>core-site.xml</h3>
fs.defaultFS: Configurado com o endereço do NameNode (hdfs://192.168.100.193:9000) para centralizar a comunicação.
Motivo: Informar o local do sistema de arquivos HDFS para todas as operações.

<h3>hdfs-site.xml</h3>
dfs.namenode.name.dir e dfs.datanode.data.dir: Diretórios locais configurados para armazenar os dados do NameNode e DataNodes.
dfs.replication: Definido como 3, para garantir redundância dos dados.
Motivo: Configurar o armazenamento físico e a replicação dos dados no cluster.

<h3>yarn-site.xml</h3>
yarn.resourcemanager.hostname: Configurado com o endereço do ResourceManager (master).
yarn.nodemanager.aux-services: Ativou o serviço de shuffle necessário para a execução de jobs no YARN.
Motivo: Permitir que o YARN coordene a execução das aplicações distribuídas.

<h3>slaves</h3>
Adicionados os endereços dos nós slave (slave1 e slave2).
Motivo: Informar ao cluster os nós que devem atuar como DataNodes e NodeManagers.
