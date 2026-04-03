# Conceitos Fundamentais do Apache Spark

# 1. Driver

O **Driver** é o processo central da aplicação Spark.

## Explicação avançada

* Responsável por:

  * Construir o **DAG (Directed Acyclic Graph)** das operações
  * Otimizar o plano de execução (Catalyst Optimizer)
  * Dividir o job em **stages** e **tasks**
  * Agendar e distribuir tarefas para os executors
  * Coletar e consolidar resultados

## Onde roda

* Local → sua máquina
* Docker → container `spark-master`
* Cloud → nó dedicado do cluster

É o **cérebro da aplicação**

---

# 2. Cluster

Um **Cluster** é o conjunto de recursos computacionais usados pelo Spark.

## ⚙️ Explicação avançada

* Composto por:

  * Driver node
  * Worker nodes
  * Cluster manager
* Responsável por executar o processamento distribuído

## Variação por ambiente

* Local → simulado (uma máquina)
* Docker → containers
* Cloud → múltiplas máquinas reais

É a **infraestrutura de execução**

---

# 3. Executor

O **Executor** é o componente que executa as tarefas distribuídas.

## ⚙️ Explicação avançada

* Roda dentro de cada nó worker
* Responsável por:

  * Executar **tasks**
  * Processar partições de dados
  * Armazenar dados em memória (cache)
* Cada executor possui:

  * CPU (cores)
  * memória dedicada

É o **motor de execução**

---

# 4. Nó (Node)

Um **Nó** é uma unidade computacional do cluster.

## ⚙️ Explicação avançada

* Pode ser:

  * Máquina física
  * Máquina virtual (cloud)
  * Container (Docker)
* Pode hospedar:

  * Driver (nó driver)
  * Executor (nó worker)

É o **computador do cluster**

---

# 5. Container

Um **Container** é um ambiente isolado que executa processos.

## ⚙️ Explicação avançada

* Criado via Docker
* Contém:

  * sistema de arquivos isolado
  * dependências
  * processo Spark (master ou worker)
* No Spark:

  * cada container pode representar um **nó**

É uma **simulação leve de máquina**

---

# 6. Sessão (SparkSession)

A **SparkSession** é o ponto de entrada da aplicação Spark.

## Explicação avançada

* Interface principal para:

  * criação de DataFrames
  * leitura e escrita de dados
  * execução de queries
* Internamente:

  * conecta seu código ao Driver
  * inicializa o contexto Spark

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()
```

É a **porta de entrada do usuário**

---

# 7. Thread

Uma **Thread** é a menor unidade de execução dentro de um processo.

## Explicação avançada

* Threads executam tarefas em paralelo dentro de um executor
* Compartilham memória do processo
* No Spark:

  * cada task é executada por uma thread

É o **nível mais baixo de execução**

---

# 8. Relação entre os componentes

```text
Usuário → SparkSession → Driver → Cluster → Nós → Executors → Threads → Tasks
```

---

# 9. Hierarquia de execução

```text
Cluster
 └── Node
      └── Executor
           └── Thread
                └── Task
```

---

# 10. Comparação por ambiente

| Conceito | Local         | Docker    | Cloud       |
| -------- | ------------- | --------- | ----------- |
| Nó       | Máquina única | Container | VM          |
| Executor | Simulado      | Container | Distribuído |
| Cluster  | Simulado      | Simulado  | Real        |
| Escala   | Baixa         | Média     | Alta        |

---

# 11. Resumo executivo

* **Driver** → cérebro (planeja e coordena)
* **Cluster** → conjunto de recursos
* **Nó** → máquina/unidade computacional
* **Executor** → executa tarefas
* **Thread** → executa task
* **Container** → simula nó (Docker)
* **SparkSession** → interface do usuário