default-cr2nesy63suinl6pmjmzicbv


# **Tecnologias Avançadas - Exercício de MapReduce**
Trabalho da discilina "Tecnologias Avançadas" parte do curso de pós graduação em "MBA em Ciências de Dados e Big Data Analytics" da Universidade Estácio

O objetivo desde trabalho é fazer um MapReduce na linguagem Java e disponibilizar os resultados e como reproduzir os resultados. Como prática deste MapReduce, vamos utilizar um txt composto de um artigo do Wikipedia e vamos fazer um contador de palavras neste texto.

Neste repositório você encontrará:
- preparação de pré-requisitos
- configuração do cluster Hadoop
- MapReduce em Java

## **Pré-Requisitos (para um ambiente Linux)**

Para ter essa demo funcional, é importante garantir que você tenha em seu ambiente o OpenJDK (Java Development Kit) e o Hadoop. Para isso, siga as instruções abaixo:

**1) instalação do OpenJDK:**
- `tabathakwk@hadoop-instance:~$ sudo apt update`
- `tabathakwk@hadoop-instance:~$ sudo apt install openjdk-8-jdk -y`
- Verifique que a instalação ocorreu corretamente executando os comandos "`java -version; javac -version`". O resultado deve ser semelhante ao abaixo:

```
tabathakwk@hadoop-instance:~$ java -version; javac -version
openjdk version "1.8.0_265"
OpenJDK Runtime Environment (build 1.8.0_265-8u265-b01-0ubuntu2~18.04-b01)
OpenJDK 64-Bit Server VM (build 25.265-b01, mixed mode)
javac 1.8.0_265
```

**2) Adicione o usuário para ser usado no ambiente Hadoop:**
- `tabathakwk@hadoop-instance:~$  sudo adduser hadoop; passwd hadoop (aqui, configure uma senha para o usuário)`
- `tabathakwk@hadoop-instance:~$  su hadoop`

**3) Instalação do Hadoop no ambiente:**
- `hadoop@hadoop-instance:~$ wget https://downloads.apache.org/hadoop/common/hadoop-3.2.1/hadoop-3.2.1.tar.gz`
- `hadoop@hadoop-instance:~$ tar xzf hadoop-3.2.1.tar.gz`

## Configuração do Cluster Hadoop

**1) Configurar o Hadoop localmente**
- Você precisa configurar os arquivos `~/.bashrc`, `$HADOOP_HOME/etc/hadoop/hadoop-env.sh`, `$HADOOP_HOME/etc/hadoop/core-site.xml`, `$HADOOP_HOME/etc/hadoop/hdfs-site.xml`, `$HADOOP_HOME/etc/hadoop/mapred-site-xml` e `$HADOOP_HOME/etc/hadoop/yarn-site.xml`
- Você pode encontrar os arquivos já configurados na pasta config_files/ -- basta copiar para a sua máquina, validar as informações (de path, etc) e copiar nos seus destinos

**2) Formatar o namenode para utilização pelo Hadoop**
- `hadoop@hadoop-instance:~$ hdfs namenode -format`

**3) Finalmente, inicie o cluster:**
- `hadoop@hadoop-instance:~$ ./hadoop-3.2.1/sbin/start-dfs.sh` # inicia o DataNode e o NameNode
- `hadoop@hadoop-instance:~$ ./hadoop-3.2.1/sbin/start-yarn.sh` # inicia os recursos yarn e o nodemgr
- para validar que os serviços subiram corretamente, rode o comando "`jps`" - o resultado deve ser semelhante a:

**hadoop@hadoop-instance:~$ jps**
```
9729 ResourceManager
9477 SecondaryNameNode
9909 NodeManager
9206 DataNode
9032 NameNode
10233 Jps
```

## MapReduce em Java

Para realizar o MapReduce em Java, você vai precisar:
- do dataset que você quer utilizar
- do código de Mapping e Reducing
- executar o código no ambiente Hadoop

**1) Criação do dataset**

Como dataset, vamos utilizar a página da Estácio no wikipedia (que pode ser encontrada aqui: https://pt.wikipedia.org/wiki/Universidade_Est%C3%A1cio_de_S%C3%A1). O conteúdo da página foi copiado e salvo no arquivo de texto "dataset/EstacioWiki.txt".

Para ter o dataset disponível no cluster Hadoop você deve seguir os seguintes passos:

a) Copie o arquivo de texto para o HDFS
- `hadoop@hadoop-instance:~/dataset$ hadoop fs -put /home/hadoop/dataset/EstacioWiki.txt /dataset.txt`

b) Verifique se o arquivo se encontra no HDFS
- `hadoop@hadoop-instance:~/dataset$ hdfs dfs -ls /` # o resultado deve ser semelhante ao abaixo:
```
Found 1 item
-rw-r--r--   1 hadoop supergroup      20009 2020-09-02 18:44 /dataset.txt
```

**2) Código Java do MapReduce**

Nesta seção, o código do exemplo é criado, compilado e compactado em formato Jar, para futura utilização no ambiente Hadoop.

a) No diretório `code/WordCount.java` você encontra o código sendo utilizado nesse exemplo

b) Foi criado um arquivo de manifesto (`code/WordCount.mf`) para facilitar a criação do arquivo Jar do exemplo

c) O código deve ser compilado e compactado em Jar:
- `hadoop@hadoop-instance:~/code$ javac WordCount.java -cp $(hadoop classpath)` # compilação do código
- `hadoop@hadoop-instance:~/code$ jar cmf WordCount.mf runner.jar 'WordCount$EstacioMapper.class' 'WordCount$MyReducer.class' WordCount.class` # compactação em formato Jar

**3) Execução do programa no Hadoop**

Uma vez que a estrutura do programa de MapReduce foi criada, ela pode ser utilizada no Hadoop conforme abaixo:
- `hadoop@hadoop-instance:~/code$ hadoop jar /home/hadoop/code/runner.jar /dataset.txt /output` # apontando para o arquivo Jar criado e salvando os outputs no diretório /output dentro do HDFS
- Uma vez que a execução do código termine, você pode checar os arquivos que foram criados no HDFS: # o arquivo /output/part-r-00000 tem o resultado da execução

```
hadoop@hadoop-instance:~/code$ hdfs dfs -ls /output
Found 2 items
-rw-r--r--   1 hadoop supergroup          0 2020-09-02 20:15 /output/_SUCCESS
-rw-r--r--   1 hadoop supergroup      14448 2020-09-02 20:15 /output/part-r-00000
```

- Para ver o resultado da execução, você roda o comando abaixo e tem um resultado semelhante ao abaixo:

```
hadoop@hadoop-instance:~/code$ hdfs dfs -cat /output/part-r-00000
2020-09-02 20:17:29,218 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
	64
"Educar	1
"Estácio"),	1
"Reclame	1
"Univerçidade,	1
"abre	1
"first"	4
"não	1
"pesquisa	1
(11	1
(17	1
(19	1
(1972);	1
(1973),	1
(1975),	2
(1976),	1
(1981)	1
(1983).	1
(2019)	1
(26	3
<...>
```

Esta etapa finaliza este trabalho.

