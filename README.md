# Explorando-Workflows-Automatizados-com-AWS-Step-Functions
Documentação da prática de automação de workflows Serverless na AWS utilizando Step Functions, com foco em lógica de estados e tratamento de erros.

**Autor:** Bruna Lima Prado
**Linkedin:** www.linkedin.com/in/brunalimaprado

## 🌟 Introdução e Objetivo

Este repositório documenta a conclusão da atividade prática do bootcamp **CodeGirls DIO em parceria com a AWS** com foco na aplicação e consolidação de **Workflows Automatizados utilizando o AWS Step Functions**.

O objetivo principal foi **orquestrar um fluxo de trabalho (State Machine)** complexo capaz de simular um processo de ingestão, paginação e processamento de dados em larga escala, utilizando integrações nativas com serviços AWS. O foco técnico reside na utilização do estado **Distributed Map** para processamento paralelo e na implementação de lógica condicional com o estado **Choice**.

## 📊 Arquitetura do Workflow (State Machine)

A Step Function testada, denominada `NOAAWeatherStateMachine`, demonstra um padrão comum de processamento de dados na AWS, orquestrando S3 e Lambda.

### Diagrama da Máquina de Estados

![Diagrama do Workflow](images/stepfunctions02.png)

### Detalhamento Técnico dos Estados

| Estado | Tipo | Ação Principal | Conceito Demonstração |
| :--- | :--- | :--- | :--- |
| **`CheckBucketContents`** | AWS S3: ListObjectsV2 | Verifica a existência inicial de objetos no *bucket* de destino. | Ponto de controle inicial. |
| **`Choice`** | Escolha | Direciona o fluxo com base no resultado da listagem (por exemplo, se o *bucket* está vazio ou não). | Lógica Condicional. |
| **`ListObjectsInNoaBucket`** | AWS S3: ListObjectsV2 | Lista os objetos no *bucket* de origem NOAA para processamento. | Preparação do *Input* para o Map. |
| **`Map`** | Mapeamento Distribuído | **Processamento em Massa e Paralelo.** Itera sobre a lista de objetos, executando o sub-fluxo para cada item. | **Escalabilidade e Fan-Out (Distribuição de Tarefas).** |
| **`CopyObject`** | AWS S3: CopyObject | Realiza a cópia de cada objeto individual da origem para o destino. | Operação de Transformação/Movimentação. |
| **`Has next page?`** | Escolha | Verifica o token de continuação (`$.States.Input.NextContinuationToken`) retornado pela operação S3. | **Implementação de Paginação** para listas S3 extensas. |
| **`ProcessNOAAData`** | AWS Lambda: Invoke | Invoca a lógica de negócio para processar os dados recém-copiados (por exemplo, leitura, filtragem). | Computação *Serverless* para processamento de objetos. |
| **`Reducer`** | AWS Lambda: Invoke | Agrega e finaliza o resultado de todas as execuções paralelas. | **Fan-In (Agregação de Resultados).** |

## 💡 Insights Adquiridos

* **Orquestração vs. Codificação:** O Step Functions permite modelar o fluxo de trabalho visualmente, substituindo a complexa lógica de *retry*, *error handling* e controle de estados que seria necessária em código (ex: Lambda).
* **Processamento de Big Data:** A utilização do estado **`Map` no modo Distribuído** é o ponto alto, permitindo a execução de milhares de cópias de objetos S3 simultaneamente, gerenciando *timeouts* e falhas de forma autônoma.
* **Tratamento de Paginação:** O uso do estado **`Choice`** em um *loop* (voltando para `ListObjectsInNoaBucket with pagi`) é a maneira eficiente de lidar com listas S3 que excedem o limite de 1000 objetos por chamada (paginação).

## ⚙️ Detalhes da Execução de Teste

A execução de teste comprovou o sucesso do *workflow* e a correta transição de estados.

### Detalhes da Execução

![Detalhes da Execução](images/stepfunctions01.png)

* **ID da Execução:** `a10cfd5c-9e27-4f95-b1e0-98d663670be5`
* **Status Final:** **Com êxito**
* **Duração:** **00:00:23.259** (Comprovando a eficiência da orquestração e paralelização).
* **Tipo:** Máquina de Estados **Standard**.

### Entrada e Saída (`Input`/`Output`)

A verificação do **Input** e **Output** valida o fluxo de dados entre os estados.

#### Entrada (`Input`)
```json
{
  "year": 2025
}

A entrada JSON simples iniciou o workflow, servindo como parâmetro para as operações subsequentes.

Saída (Output)
O resultado final encapsula o payload processado e metadados de execução, confirmando o tratamento de dados (como o year: 2025) e a correta passagem de informações HTTP/AWS entre os serviços.

🛠️ **Tecnologias Chave**

**AWS Step Functions:** Orquestração serverless baseada na linguagem ASL (Amazon States Language).

**AWS S3:** Serviços de armazenamento e manipulação de objetos (ListObjectsV2, CopyObject).

**AWS Lambda:** Lógica de negócio e processamento de dados serverless (ProcessNOAAData, Reducer).

**IAM:** Definição do Perfil de Execução (Role) para garantir o Princípio do Menor Privilégio na interação entre os serviços.
