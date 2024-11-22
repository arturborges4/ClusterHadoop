# ClusterHadoop no HyperVisor Proxmox
Cluster Hadoop utilizando VMs Ubuntu Server no Proxmox.


🖥️Configuração do Cluster

Foi utilizado o HyperVisor Proxmox rodando em um servidor para provisionar maquinas virtuais (VMs), instalado o Ubuntu Server na sua versão 20.04 (afim de economizar recursos, não foi usada a ultima versão do Ubuntu), o Java JDK 8 e o Hadoop 3.3.6. 
Configuração da maquina Host:
 - Processador FX8300 8 Cores 3.3 GHz base core.
 - Memória RAM 3.6 GBs DDR3.
 - HDD 1TB.

![Servidor_resized](https://github.com/user-attachments/assets/0bdc4c49-3a56-4c06-ad3a-360547294418)

Definidos os IPs estáticos para as máquinas diretamente nas configurações do roteador.

![IPSestaticosROTEADOR_resized_resized](https://github.com/user-attachments/assets/406b2732-d98d-4a63-98d3-5b665c874536)


Após as configurações, o nó Master passou a identificar os nós Slaves.

![datanodes3](https://github.com/user-attachments/assets/151d16f0-bd84-46f2-9315-52ab14f83218)

![datanodesWEBGUI](https://github.com/user-attachments/assets/2813e648-d57d-4707-846a-0b39dc9ace06)

Na interface WEB do Proxmox foi possivel acompanhar o consumo de recursos do servidor. Destaque para a limitação de memória com as três máquinas rodando simultaneamente.

![proxmox](https://github.com/user-attachments/assets/919f4899-9212-41c7-a7b1-92fd8ad29e03)


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

- Criado um arquivo input.txt contendo palavras simples.
- Usado o comando ```hadoop fs -mkdir``` para criar um diretório de entrada no HDFS.
- Carregado o arquivo input.txt no HDFS com ```hadoop fs -put```
- Executado o job WordCount usando:
```hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar wordcount /user/hadoop/input /user/hadoop/output```
- Verificação dos resultados com ```hadoop fs -cat /user/hadoop/output/part-r-00000```


<h3>2. Onde o arquivo foi enviado e como foi especificado:</h3>
O arquivo input.txt foi enviado ao HDFS no diretório /user/hadoop/input.
Na execução do WordCount, o diretório de entrada foi especificado como /user/hadoop/input, e o de saída foi definido como /user/hadoop/output.

<h3>3. Como o Hadoop divide e distribui as tarefas:</h3>

- O HDFS divide o arquivo de entrada em blocos (padrão: 128 MB).

- Cada bloco é processado por um Map Task, atribuído aos DataNodes que armazenam os blocos.

- O Reduce Task combina os resultados intermediários dos Map Tasks, produzindo a saída final.



<h1>Análise de Desempenho</h1>

<h3>1. Tempo para processar e fatores que influenciam o desempenho: </h3>

- O processamento foi prejudicado pelas limitações de Hardware da máquina host, que contava com apenas um processador AMD fx8300 de 8 cores, 3.6 GBs DDR3 de memória RAM e um disco rígido HDD. A baixa disponibilidade de memória foi um fator crucial, pois gerou a necessidade de alocar apenas 1GB e 2 cores para cada nó do cluster, já que o próprio HyperVisor Proxmox necessita de recursos para trabalhar. 


<h3>2. Distribuição da carga entre os nós:</h3>

- O job foi distribuído entre os três DataNodes, com cada nó processando blocos armazenados localmente.

- Razões para carga desigual:

     - Dados locais: O Hadoop tenta processar blocos no mesmo nó onde estão armazenados.
     
     - Número de blocos: Se o arquivo possui poucos blocos, nem todos os nós serão usados igualmente.    

     - Configuração do cluster: Parâmetros como número de Map/Reduce Tasks podem limitar a paralelização.



<h1>Desafios e Resolução de Problemas</h1>

<h3>1. Desafios enfrentados e soluções aplicadas:</h3>

- Desafio 1: Instalação das versões corretas.
     - O primeiro problema enfrentado foi a instalação do Java JDK. A Hadoop é projetado para trabalhar com a versão 8 do Java na build escolhida (3.3.6). Ao tentar rodar com o Java 11, problemas de compatibilidade dificultaram a configuração. O reset da VM para    
     instalação do Java 8 permitiu prosseguir.


- Desafio 2: Problemas de conectividade entre os nós.
     - Sintoma: Os slaves não eram reconhecidos pelo master, resultando em falhas na execução de jobs.
     - Causa: Endereços IP dinâmicos atribuídos às VMs alteravam a comunicação entre os nós.
     - Solução: Configurado IPs estáticos para todas as VMs e atualizado o arquivo hosts para associar nomes e IPs.



- Desafio 3: Diretórios de armazenamento ausentes.
     - Sintoma: Erro ao inicializar o NameNode/DataNodes devido à falta dos diretórios configurados.
     - Causa: Diretórios especificados nos arquivos de configuração não haviam sido criados manualmente.
     - Solução: Criei os diretórios especificados em dfs.namenode.name.dir e dfs.datanode.data.dir e ajustei permissões com chown.
 

- Desafio 4: Erros de permissões no HDFS.
     - Sintoma: Impossibilidade de carregar arquivos para o HDFS devido a erros de acesso.
     - Causa: O usuário do sistema não tinha permissões adequadas para operar no HDFS.
     - Solução: Ajustei permissões usando hdfs dfs -chmod e configurei o ambiente para que o usuário artur fosse o proprietário do HDFS.
 

- Desafio 5: Lerdeza na execução de jobs.
     - Sintoma: Jobs simples demoravam mais do que o esperado.
     - Causa: Recursos insuficientes (memória e CPU) disponíveis nas VMs.
     - Solução: Rdistribuido os recursos alocados no Proxmox, garantindo 1 GB por nó e otimizado o tamanho do arquivo de entrada para testes.

 
- Desafio 6: Arquivo de configuração inválido.
     - Sintoma: Erro ao iniciar o cluster devido a configurações incorretas nos arquivos XML.
     - Causa: Sintaxe inválida (ex.: tags não fechadas) ou valores inconsistentes.
     - Solução: Validação dos arquivos com atenção à sintaxe e utilizados exemplos de configuração como referência.



<h3>2. Problemas comuns na configuração de um cluster Hadoop e como resolvê-los:</h3>
- Problema 1: Comunicação entre nós.
     - Causa: DNS mal configurado ou falta de IPs fixos.
     - Dica: Configure o arquivo /etc/hosts para associar nomes e IPs de todos os nós. Use IPs estáticos para evitar inconsistências.


- Problema 2: Diretórios do HDFS ausentes ou incorretos.
     - Causa: Diretórios especificados nos arquivos de configuração não existem.
     - Dica: Crie manualmente os diretórios configurados em hdfs-site.xml e defina permissões corretas.

 
- Problema 3: Erros ao iniciar o YARN.
     - Causa: Falta de parâmetros necessários no yarn-site.xml.
     - Dica: Verifique se ```yarn.resourcemanager.hostname``` e ```yarn.nodemanager.aux-services``` estão configurados corretamente.
 
- Problema 4: Falha ao carregar dados no HDFS.
     - Causa: Usuário do sistema não tem permissões adequadas.
     - Dica: Certifique-se de que o usuário correto seja proprietário do HDFS e configure permissões adequadas com hdfs dfs -chmod.
 
- Problema 5: Jobs MapReduce não distribuídos.
     - Causa: Arquivo de entrada pequeno ou configuração de tarefas limitada.
     - Dica: Use arquivos maiores para testar a distribuição e ajuste os parâmetros de Map/Reduce Tasks.
 

LOGS WORDCOUNT:

![wordcount-teste-3nos](https://github.com/user-attachments/assets/47e45756-a355-4c62-9263-d98b53f4d022)
