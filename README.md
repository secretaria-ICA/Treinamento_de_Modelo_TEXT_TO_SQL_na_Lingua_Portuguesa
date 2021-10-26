## Projeto Final do Curso CCE PUC-Rio - BI MASTER 2019.2

### Treinamento de modelo TEXT-TO-SQL na língua portuguesa

**Aluno: [Allan Miranda](https://github.com/allangxg) - Matrícula: 192.190.012**

**Orientador: Leonardo Mendonça**

**Trabalho apresentado ao curso [BI MASTER](https://ica.puc-rio.ai/es/bi-master-es/) como pré-requisito para conclusão de "Curso de Pós Graduação Business Intelligence Master" na Pontifícia Universidade Católica do Rio de Janeiro**


### **Resumo**

Um desafio em aberto na área de processamento de linguagem natural e que vem atraindo muito interesse da comunidade é permitir buscar respostas em um banco de dados, convertendo o texto de entrada em consulta SQL estruturada. É possível encontrar na literatura bastante avanço de pesquisas no idioma inglês, porém temos poucas iniciativas no nosso querido português.

Recentemente, foi publicado um artigo por pesquisadores brasileiros - intitulado _mRAT-SQL+GAP: A Portuguese Text-to-SQL Transformer*_ e que pode ser encontrado [aqui](https://arxiv.org/abs/2110.03546) - que treinaram um modelo adaptado do sistema RAT-SQL + GAP com o desafio de investigar a tradução para SQL quando a pergunta de entrada é fornecida na língua portuguesa.

Esse experimento serviu como inspiração desse trabalho, onde pretendo utilizar uma outra abordagem para treinamento do modelo na língua portuguesa. Para isso, irei utilizar a arquitetura descrita no artigo _Bridging Textual and Tabular Data for Cross-Domain Text-to-SQL Semantic Parsing_, cujo documento pode ser encontrado [aqui](https://arxiv.org/abs/2012.12627).

---
### **1. Introdução**

A grande maioria das empresas utilizam bancos de dados relacionais, porém sabemos que as formas tradicionais de interface com o banco, utilizando SQL, não são intuitivas para usuários não especialistas no assunto. Imagine como pode ser vantajoso para as empresas ter a capacidade de entregar o poder de realizar consultas no banco de dados ao time de negócio, com grande capacidade de extrair insights importantes sobre os dados, mas que hoje não são capazes por falta de conhecimento técnico.

Diante desse dasafio, começaram a surgir modelos de aprendizagem de máquina capazes de traduzir a linguagem natural para consultas de banco de dados. Hoje, conseguimos encontrar vários modelos dispostos a resolver essa dor, porém a maioria deles avançaram somente no idiona inglês. 

(continua... em construção)


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

    %cd '/home/jupyter/TabularSemanticParsing'
    !python3 data/spider/scripts/amend_missing_foreign_keys.py data/spider
    !./experiment-bridge.sh configs/bridge/spider-bridge-bert-large.sh --process_data 0

**2.3.4: Treinamento**

    !./experiment-bridge.sh configs/bridge/spider-bridge-bert-large.sh --train 0

---

### **3. Resultados**

Todos os detalhes da execução do modelo pode ser encontrado [aqui](https://wandb.ai/allangxg/smore-spider-group--final/groups/ppl-0.85.2.lstm.meta.ts.bert-large-uncased-norm-digit-400-0-0.0-0.0005-inverse-square-inverse-square-100000-4000?workspace=user-allangxg).

![Treinamento](https://github.com/allangxg/nl2sql/blob/main/treinamento.png)

![Fine Tuning Rate](https://github.com/allangxg/nl2sql/blob/main/finetuning.png)

![Learning Rate](https://github.com/allangxg/nl2sql/blob/main/learningrate.png)

---

### **4. Conclusão**

---
