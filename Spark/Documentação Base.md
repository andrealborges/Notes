# Apache Spark

# 1. Introdução ao Spark

O Spark é um framework de processamento distribuído para grandes volumes de dados (Big Data).

## Principais características
- Processamento distribuído
- Execução em memória (in-memory)
- Escalabilidade horizontal
- Suporte a múltiplas linguagens

## Spark vs processamento tradicional

| Aspecto | Tradicional | Spark |
|--------|------------|------|
| Execução | Sequencial | Paralela |
| Escala | Limitada | Distribuída |
| Performance | Baixa | Alta |

---

# 2. Arquitetura do Spark

## Componentes principais

### Driver
- Responsável por coordenar a execução
- Cria o plano de execução (DAG)

### Executor
- Executa tarefas
- Processa dados distribuídos

### Cluster Manager
- Gerencia recursos do cluster

---

## Fluxo de execução

1. Usuário envia código
2. Driver cria um Job
3. Job é dividido em Stages
4. Stages são divididos em Tasks
5. Tasks são executadas nos Executors

---

# 3. APIs do Spark

## APIs disponíveis

- RDD (baixo nível)
- DataFrame (principal)
- Dataset (tipado)

## Linguagens

- Python (PySpark)
- Scala
- SQL

---

# 4. Lazy Evaluation

O Spark utiliza avaliação preguiçosa:

- Transformações não executam imediatamente
- Execução ocorre apenas em ações

---

# 5. Spark em Cloud

## Plataformas principais

- Databricks
- Fabrick
- AWS

## Como o Spark funciona na nuvem

Diferente do ambiente local, na cloud:

Você NÃO gerencia diretamente:
- Máquinas
- Cluster físico
- Configuração de rede

A plataforma gerencia automaticamente:
- Provisionamento de recursos
- Escalabilidade
- Execução distribuída

## Componentes na Cloud

### Cluster gerenciado
- Criado sob demanda
- Pode ser auto-scale
- Contém driver + executors

### Storage desacoplado
- Data Lake (ex: ADLS, S3)
- Dados persistidos fora do cluster

### Notebook/Job
- Interface de execução
- Código enviado para o cluster


## Fluxo de execução na cloud

1. Usuário executa notebook
2. Código é enviado ao cluster
3. Driver é iniciado
4. Executors são provisionados
5. Dados são lidos do Data Lake
6. Processamento ocorre distribuído
7. Resultado é salvo no storage

## Vantagens

- Escalabilidade automática
- Alta disponibilidade
- Integração com ferramentas (ML, BI)

## Cuidados

- Custo (clusters ativos)
- Dependências não versionadas
- Diferença entre ambientes (dev vs prod)

---

# 5. Spark na Máquina Local

## Quando usar

- Desenvolvimento
- Testes
- Debug

## Como funciona

- Tudo roda em UMA máquina
- Spark simula paralelismo via threads
- Não há cluster real

## Instalação

### Pré-requisitos
- Python 3.x
- Java (JDK 8+)

### Instalar PySpark

```bash
pip install pyspark
```

## Criar sessão Spark

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("MeuApp") \
    .master("local[*]") \
    .getOrCreate()
```

## Explicação do `local[*]`

* `local` → execução local
* `*` → usa todos os núcleos da máquina

## Limitações

* Sem distribuição real
* Limitado por CPU/RAM local
* Não representa comportamento de cluster

## Diferenças vs Cloud

| Aspecto     | Local    | Cloud |
| ----------- | -------- | ----- |
| Escala      | Limitada | Alta  |
| Cluster     | Simulado | Real  |
| Performance | Menor    | Alta  |

---

# 6. Spark com Docker

## Por que usar Docker?

* Padronizar ambiente
* Evitar problemas de instalação
* Simular cluster local
* Reprodutibilidade

## Conceito

Com Docker, você roda:

* Spark Master (container)
* Spark Worker(s) (containers)

---

## Exemplo docker-compose

```yaml
version: '3'

services:
  spark-master:
    image: bitnami/spark:latest
    environment:
      - SPARK_MODE=master
    ports:
      - "8080:8080"
      - "7077:7077"

  spark-worker:
    image: bitnami/spark:latest
    environment:
      - SPARK_MODE=worker
      - SPARK_MASTER_URL=spark://spark-master:7077
```

## Subir cluster

```bash
docker-compose up -d
```

## Interfaces

* Master UI → [http://localhost:8080](http://localhost:8080)

## Como funciona

1. Container master inicia
2. Workers conectam ao master
3. Spark distribui tarefas
4. Execução ocorre entre containers

## Diferença importante

| Ambiente   | Execução                      |
| ---------- | ----------------------------- |
| Local puro | Threads                       |
| Docker     | Containers (cluster simulado) |
| Cloud      | Cluster real                  |

## Vantagens

* Ambiente reproduzível
* Próximo do ambiente real
* Ideal para testes

## Cuidados

* Uso de memória
* Configuração de rede
* Persistência de dados

---

# 7. Spark Standalone Cluster (Local)

* Master + Workers
* Pode ser rodado via Docker ou instalação manual
* Simula ambiente distribuído

---

# 8. Visualização

## Local
```text
┌─────────────────────────────────────────────┐
│                 SUA MÁQUINA                 │
│                                             │
│  ┌──────────────┐                           │
│  │   DRIVER     │  ← cérebro                │
│  └──────┬───────┘                           │
│         │                                   │
│  ┌──────▼────────┐                          │
│  │  EXECUTOR     │  ← simulado              │
│  └──────┬────────┘                          │
│         │                                   │
│   ┌─────▼─────┐ ┌─────▼─────┐ ┌─────▼─────┐ │
│   │ Thread 1  │ │ Thread 2  │ │ Thread 3  │ │
│   └───────────┘ └───────────┘ └───────────┘ │
│                                             │
└─────────────────────────────────────────────┘
```

## Máquina

```text
                ┌──────────────────────┐
                │     DRIVER NODE      │
                │   (cérebro do job)   │
                └──────────┬───────────┘
                           │
                           ▼
        ┌───────────────────────────────────────┐
        │           CLUSTER MANAGER             │
        │ (gerencia recursos e distribuição)    │
        └──────────┬──────────┬──────────┬──────┘
                   │          │          │
                   ▼          ▼          ▼

        ┌────────────┐ ┌────────────┐ ┌────────────┐
        │   NODE 1   │ │   NODE 2   │ │   NODE 3   │
        │ (Worker)   │ │ (Worker)   │ │ (Worker)   │
        └────┬───────┘ └────┬───────┘ └────┬───────┘
             │              │              │
     ┌───────▼──────┐ ┌────▼────────┐ ┌───▼─────────┐
     │  EXECUTOR    │ │  EXECUTOR   │ │  EXECUTOR   │
     └────┬─────────┘ └────┬────────┘ └────┬────────┘
          │                │               │
   ┌──────▼─────┐   ┌──────▼─────┐   ┌─────▼──────┐
   │ Threads     │   │ Threads     │   │ Threads     │
   │ (Tasks)     │   │ (Tasks)     │   │ (Tasks)     │
   └─────────────┘   └─────────────┘   └─────────────┘
```

## Docker

```text
                 ┌────────────────────────────┐
                 │        DOCKER HOST         │
                 │    (sua máquina física)    │
                 └─────────────┬──────────────┘
                               │
        ┌──────────────────────┴──────────────────────┐
        │                                             │

┌──────────────────────┐                  ┌──────────────────────┐
│    spark-master      │                  │    spark-worker      │
│  (Driver + Master)   │                  │     (Executor)       │
│                      │                  │                      │
│  - Planeja execução  │                  │  - Executa tasks     │
│  - Gerencia cluster  │                  │  - Processa dados    │
└──────────┬───────────┘                  └──────────┬───────────┘
           │                                         │
           │                                         │
           │                  ┌──────────────────────▼──────────────┐
           │                  │          spark-worker               │
           │                  │           (Executor)               │
           │                  │                                    │
           │                  │   - Executa tasks                  │
           │                  │   - Processa dados                 │
           │                  └──────────┬─────────────────────────┘
           │                             │
           ▼                             ▼

   ┌──────────────┐               ┌──────────────┐
   │  Threads     │               │  Threads     │
   │  (Tasks)     │               │  (Tasks)     │
   └──────────────┘               └──────────────┘
```

```text
            DRIVER (spark-master)
                     ↓
        ┌────────────┼────────────┐
        ↓            ↓            ↓
   WORKER       WORKER       WORKER
 (container)  (container)  (container)
        ↓            ↓            ↓
     Threads      Threads      Threads
```