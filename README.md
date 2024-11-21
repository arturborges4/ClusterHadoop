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



<h1>2. Configurações alteradas nos arquivos do Hadoop e motivos:</h1>

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



<h1>Execução da Aplicação</h1>
<h3>1. Etapas para executar o WordCount:</h3>

- Criei um arquivo input.txt contendo palavras simples.
- Usei o comando ```hadoop fs -mkdir``` para criar um diretório de entrada no HDFS.
- Carreguei o arquivo input.txt no HDFS com ```hadoop fs -put```
- Executei o job WordCount usando:
```hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar wordcount /user/hadoop/input /user/hadoop/output```
- Verifiquei os resultados com ```hadoop fs -cat /user/hadoop/output/part-r-00000```


<h3>2. Onde o arquivo foi enviado e como foi especificado:</h3>
O arquivo input.txt foi enviado ao HDFS no diretório /user/hadoop/input.
Na execução do WordCount, o diretório de entrada foi especificado como /user/hadoop/input, e o de saída foi definido como /user/hadoop/output.

<h3>3. Como o Hadoop divide e distribui as tarefas:</h3>
- O HDFS divide o arquivo de entrada em blocos (padrão: 128 MB).
- Cada bloco é processado por um Map Task, atribuído aos DataNodes que armazenam os blocos.
- O Reduce Task combina os resultados intermediários dos Map Tasks, produzindo a saída final.


<h3>2. Distribuição da carga entre os nós:</h3>
- O job foi distribuído entre os três DataNodes, com cada nó processando blocos armazenados localmente.
- Razões para carga desigual
     Dados locais: O Hadoop tenta processar blocos no mesmo nó onde estão armazenados.                                                                                                                                                       
     Número de blocos: Se o arquivo possui poucos blocos, nem todos os nós serão usados igualmente.                                                                                                                            
     Configuração do cluster: Parâmetros como número de Map/Reduce Tasks podem limitar a paralelização.
