## Projeto Final do Curso CCE PUC-Rio - BI MASTER 2019.2

### Treinamento de modelo TEXT-TO-SQL na língua portuguesa

**Aluno: [Allan Miranda](https://github.com/allangxg) - Matrícula: 192.190.012**

**Orientador: Leonardo Mendonça**

**Trabalho apresentado ao curso [BI MASTER](https://ica.puc-rio.ai/es/bi-master-es/) como pré-requisito para conclusão de "Curso de Pós Graduação Business Intelligence Master" na Pontifícia Universidade Católica do Rio de Janeiro**


### **Resumo**

Um desafio em aberto na área de processamento de linguagem natural e que vem atraindo muito interesse da comunidade é permitir buscar respostas em um banco de dados, convertendo o texto de entrada em consulta SQL estruturada. É possível encontrar na literatura bastante avanço de pesquisas no idioma inglês, porém temos poucas iniciativas no nosso querido português.

Recentemente, foi publicado um artigo por pesquisadores brasileiros - intitulado _mRAT-SQL+GAP: A Portuguese Text-to-SQL Transformer*_ e que pode ser encontrado [aqui](https://arxiv.org/abs/2110.03546) - onde treinaram um modelo adaptado do sistema RAT-SQL + GAP com o desafio de investigar a tradução para SQL quando a pergunta de entrada é fornecida na língua portuguesa.

Esse experimento serviu como inspiração desse trabalho, onde pretendo utilizar uma outra abordagem para treinamento do modelo na língua portuguesa. Para isso, irei utilizar a arquitetura descrita no artigo _Bridging Textual and Tabular Data for Cross-Domain Text-to-SQL Semantic Parsing_, cujo documento pode ser encontrado [aqui](https://arxiv.org/abs/2012.12627).

---
### **1. Introdução**

A grande maioria das empresas utilizam bancos de dados relacionais, porém sabemos que as formas tradicionais de interface com o banco, utilizando SQL, não são intuitivas para usuários não especialistas no assunto. Imagine como pode ser vantajoso para as empresas ter a capacidade de entregar o poder de realizar consultas no banco de dados ao time de negócio, com grande capacidade de extrair insights importantes sobre os dados, mas que hoje não são capazes por falta de conhecimento técnico.

Diante desse dasafio, começaram a surgir modelos de aprendizagem de máquina capazes de traduzir a linguagem natural para consultas de banco de dados. Hoje, conseguimos encontrar vários modelos dispostos a resolver essa dor, porém a maioria deles avançaram somente no idioma inglês. 

A proposta desse trabalho é utilizar projetos criados por outros pesquisadores que se propuseram a resolver esse problema e treinar um modelo que possa realizar essa tradução a partir do idioma português.

---
### **2. Modelagem**


**2.1 Objetivo:**

O objetivo do projeto é treinar um modelo de tradução de linguagem natural em consulta SQL no idioma português.



**2.2 Premissas:**


- O treinamento será realizado utilizando o conjunto de dados [Spider](https://arxiv.org/abs/1809.08887) que foi traduzido do inglês no projeto [mRAT-SQL+GAP](https://github.com/C4AI/gap-text2sql)
- O projeto [TabularSemanticParsing](https://github.com/salesforce/TabularSemanticParsing) será adaptado para utilizar como input do treinamento o conjunto de dados em inglês e português, citado acima.



**2.3 Aplicação e codificação:**


Esse projeto foi desenvolvido usando o Jupyter Notebook e o modelo foi treinado numa instância do Google Cloud de 16 vCPUs, 104 GB RAM e 1 GPU Tesla K80.

Passo a passo dos comandos:

**2.3.1: Download do projeto mRAT-SQL+GAP**


    # Download do projeto mRAT-SQL+GAP que serão usados os arquivos traduzidos do conjunto de dados SPIDER
    !git clone https://github.com/C4AI/gap-text2sql
    %cd gap-text2sql/mrat-sql-gap

**2.3.1.1: Instalação dos pacotes**

    !conda create --name mtext2sql python=3.7 --yes
    !source activate mtext2sql
    !conda install pytorch=1.5 cudatoolkit=10.2 -c pytorch --yes
    !pip install gdown
    !pip install -r requirements.txt
    !python -c "import nltk; nltk.download('stopwords'); nltk.download('punkt')"
    !conda install -c conda-forge jupyter_contrib_nbextensions --yes

**2.3.1.2: Setup do projeto que irá realizar o download dos arquivos traduzidos**

    %%bash
    chmod +x setup.sh
    ./setup.sh

**2.3.2: Download do projeto TabularSemanticParsing**

    !git clone https://github.com/salesforce/TabularSemanticParsing
    %cd /home/jupyter/TabularSemanticParsing/

**2.3.2.1: Instalação dos pacotes**

    !pip install torch torchvision
    !python3 -m pip install -r requirements.txt
    !python3 -m pip install mo-future --upgrade
    !export PYTHONPATH=`/home/jupyter/TabularSemanticParsing` && python -m nltk.downloader punkt

**2.3.2: Setup com as modificações do nosso projeto**

    !mv '/home/jupyter/gap-text2sql/mrat-sql-gap/data/spider-en-pt/database' '/home/jupyter/TabularSemanticParsing/data/spider'
    !mv '/home/jupyter/gap-text2sql/mrat-sql-gap/data/spider-en-pt/dev.json' '/home/jupyter/TabularSemanticParsing/data/spider'
    !mv '/home/jupyter/gap-text2sql/mrat-sql-gap/data/spider-en-pt/tables.json' '/home/jupyter/TabularSemanticParsing/data/spider'
    %cd '/home/jupyter/TabularSemanticParsing/data/spider'
    !gdown --id '1Ve2i3-oxBbrV87_UqHI09q1aKjFlZrws'

**2.3.3: Pré-Processamento**

Antes de executar os passos abaixo é necessário realizar um ajuste no arquivo _TabularSemanticParsing/data/spider/scripts/amend_missing_foreign_keys.py_. Abra o arquivo e inclua o valor abaixo na primeira linha. 

    #!/usr/bin/env python

Salve e prossiga com os comandos abaixo.

    %cd '/home/jupyter/TabularSemanticParsing'
    !python3 data/spider/scripts/amend_missing_foreign_keys.py data/spider
    !./experiment-bridge.sh configs/bridge/spider-bridge-bert-large.sh --process_data 0

**2.3.4: Treinamento**

As configurações do modelo foram editadas para os valores abaixo antes de iniciar o treinamento. O arquivo de configuração pode ser encontrado nesse caminho: _TabularSemanticParsing/configs/bridge/spider-bridge-bert-large.sh_

    num_steps=400
    train_batch_size=2
    dev_batch_size=2

Após a edição, execute o comando abaixo para iniciar o treinamento.

    !./experiment-bridge.sh configs/bridge/spider-bridge-bert-large.sh --train 0

---

### **3. Resultados**

Todos os detalhes da execução do modelo podem ser encontrados [aqui](https://wandb.ai/allangxg/smore-spider-group--final).

![Treinamento](https://github.com/allangxg/nl2sql/blob/main/treinamento.png)

O treinamento do modelo foi interrompido após a execução de, aproximadamente, 400 épocas. Para ter um resultado mais preciso seria necessário aumentar esse número, o que acarretaria em um tempo de execução e custo maior.

Mas é animador o resultado que podemos obter mesmo com pouco tempo de treinamento. Veja na imagem abaixo, que apesar de criar o SQL incompleto para responder a pergunta, obtivemos o mesmo resultado quando a pergunta é feita em inglês e português.

![Inferência](https://github.com/allangxg/nl2sql/blob/main/inferencia.png)

---

### **4. Conclusão**

O trabalho teve como objetivo treinar um modelo de linguagem natural no idioma português que fosse capaz de traduzir o texto para consulta de banco de dados.

Foram estudados 3 artigos que se propuseram a resolver esse problema, porém com características distintas, são eles: [mRAT-SQL+GAP:A Portuguese Text-to-SQL Transformer](https://arxiv.org/abs/2110.03546), [Bridging Textual and Tabular Data for Cross-Domain Text-to-SQL Semantic Parsing](https://arxiv.org/abs/2012.12627) e [TaPas: Weakly Supervised Table Parsing via Pre-training](https://aclanthology.org/2020.acl-main.398/).

A decisão foi utilizar a arquitetura proposta no projeto BRIDGE com o dataset [Spider](https://yale-lily.github.io/spider) traduzido do inglês para o português apresentado no projeto mRAT-SQL+GAP. Essa decisão ocorreu por considerar que a proposta do projeto BRIDGE possui o diferencial de não só modelar o problema levando em conta os nomes das tabelas, campos e suas relações, mas também, considera os valores dos campos como _anchor text_ para relacionar os termos utilizados nas perguntas com os valores correspondentes nos campos do banco de dados.

![Representação do modelo. Imagem retirada do artigo original](https://github.com/allangxg/nl2sql/blob/main/arquitetura.png)

Foi utilizado o modelo _transformer (BERT)_ na camada de encoder combinado com uma rede _LSTM pointer-generator_ com _multi-head attention_ na camada de decoder.

O próximo objetivo será treinar o modelo usando 100 mil épocas como proposto no artigo original com o objetivo de criar um modelo mais assertivo nas traduções.

---
