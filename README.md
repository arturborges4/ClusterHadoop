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
