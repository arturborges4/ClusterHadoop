# ClusterHadoop
Cluster Hadoop utilizando VMs Ubuntu Server no Proxmox


<h1>Configuração do Cluster</h1>
1. Função de cada nó no cluster Hadoop:

<h2>NameNode (Master):</h2>

É responsável por gerenciar o metadata do sistema de arquivos distribuído HDFS (ex.: nomes dos arquivos, blocos, localizações nos DataNodes).
Coordena as operações de leitura/escrita e é essencial para a operação do cluster.

<h2>DataNode (Slaves):</h2>

Armazenam os dados reais em blocos.
Respondem às solicitações do NameNode para leitura, escrita e replicação de dados.

<2>ResourceManager (Master):</h2>

Gerencia os recursos do cluster para execução de aplicações distribuídas.
Trabalha com o NodeManager para alocar contêineres e monitorar tarefas.

<h2>NodeManager (Slaves):</h2>

Executa tarefas atribuídas pelo ResourceManager.
Monitora o uso de recursos (CPU, memória) em cada nó.
