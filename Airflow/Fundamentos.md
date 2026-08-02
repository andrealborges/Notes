# Airflow — Fundamentos (o que é e por que usar)

Este tópico apresenta a base do **Apache Airflow** de forma prática. O objetivo é responder:
> "O que exatamente o Airflow faz e por que ele é tão usado para orquestrar pipelines de dados e processos automatizados?"

Antes de entrar em instalação, DAGs e comandos, você precisa entender os conceitos que sustentam todo o uso da ferramenta.

---

## 1) O que é o Airflow?

O **Apache Airflow** é uma plataforma para **criar, agendar e monitorar workflows** (fluxos de trabalho) de forma programática.

Na prática, ele resolve um problema muito comum:
> "Eu preciso rodar uma sequência de tarefas, em uma ordem específica, em horários definidos, com visibilidade sobre o que deu certo e o que falhou."

Com Airflow, você define esses fluxos como código Python, incluindo:
- quais tarefas existem
- em que ordem elas devem rodar
- quando devem ser executadas
- o que fazer em caso de falha

Assim, processos que antes viviam em scripts soltos ou crontabs desorganizados passam a ter histórico, retries, logs e uma interface visual.

---

## 2) O que é uma DAG?

**DAG** significa **Directed Acyclic Graph** (grafo acíclico dirigido).

No Airflow, uma DAG representa um **workflow completo**: um conjunto de tarefas conectadas por dependências, sem ciclos (uma tarefa nunca depende dela mesma, direta ou indiretamente).

### Características principais
- é definida em um arquivo Python
- tem um identificador único (`dag_id`)
- possui uma frequência de execução (`schedule`)
- é composta por uma ou mais **tasks**

### Exemplo mental
Uma DAG pode representar:
- extrair dados de uma API → tratar os dados → carregar em um banco
- rodar um pipeline de ETL todo dia às 6h
- disparar um treinamento de modelo após a chegada de um arquivo

---

## 3) O que é uma Task?

Uma **task** é a menor unidade de trabalho dentro de uma DAG.

Ela representa uma ação concreta, como:
- rodar um script Python
- executar um comando bash
- esperar um arquivo aparecer
- rodar uma query em um banco

### Regra prática
- **DAG** = o workflow inteiro
- **Task** = um passo dentro desse workflow

### Exemplo
Uma DAG de ETL pode ter três tasks: `extrair`, `transformar` e `carregar`, executadas nessa ordem.

---

## 4) O que é um Operator?

Um **Operator** é o "molde" que define o que uma task efetivamente faz.

Cada task é, na prática, a instância de um Operator.

### Operators mais comuns
- **PythonOperator** → executa uma função Python
- **BashOperator** → executa um comando shell
- **EmptyOperator** → não faz nada, útil para organizar o fluxo
- **Sensors** → esperam uma condição ser satisfeita (arquivo, horário, outra DAG)
- **Operators de providers** → integração com bancos, cloud, APIs (ex.: `PostgresOperator`, `S3Hook`, etc.)

### Regra prática
- **Operator** = tipo de tarefa
- **Task** = a tarefa configurada dentro da DAG, usando um Operator

---

## 5) O que é o Scheduler?

O **Scheduler** é o componente responsável por decidir **quando** cada DAG e cada task devem rodar.

Ele fica constantemente:
- lendo as DAGs
- verificando o `schedule` de cada uma
- disparando as execuções na hora certa
- monitorando dependências entre tasks

### Regra prática
Sem o Scheduler rodando, nenhuma DAG é executada automaticamente.

---

## 6) O que é o Executor?

O **Executor** define **como e onde** as tasks são efetivamente executadas.

### Principais tipos
- **SequentialExecutor** → executa uma task por vez, sem paralelismo (uso didático/local)
- **LocalExecutor** → executa tasks em paralelo na mesma máquina
- **CeleryExecutor** → distribui tasks entre múltiplos workers
- **KubernetesExecutor** → cria um pod Kubernetes para cada task

### Regra prática
- **ambiente de estudo/teste simples** → SequentialExecutor ou LocalExecutor
- **ambiente de produção com volume** → CeleryExecutor ou KubernetesExecutor

---

## 7) O que é o Webserver?

O **Webserver** é a interface visual (UI) do Airflow.

Nela você consegue:
- ver todas as DAGs cadastradas
- acompanhar execuções em tempo real
- visualizar logs de cada task
- disparar execuções manuais
- pausar/despausar DAGs

Essa UI é um dos grandes motivos do Airflow ser tão popular: dá visibilidade real sobre pipelines que antes eram "caixas-pretas".

---

## 8) O que é o Metadata Database?

O **Metadata Database** é o banco de dados onde o Airflow guarda **todo o estado** do sistema:
- DAGs conhecidas
- histórico de execuções
- status de cada task
- variáveis e conexões configuradas
- usuários da UI

### Regra prática
Sem esse banco funcionando corretamente, o Airflow perde a capacidade de saber o que já rodou, o que está rodando e o que falhou.

Em ambientes locais simples, normalmente é SQLite. Em ambientes reais, normalmente é PostgreSQL ou MySQL.

---

## 9) Principais conceitos do Airflow (resumo)

Para usar Airflow bem, você precisa dominar estes conceitos:

### DAG
Workflow completo, definido como código Python.

### Task
Unidade individual de trabalho dentro da DAG.

### Operator
Tipo de tarefa (o que a task efetivamente faz).

### Scheduler
Decide quando cada DAG/task deve rodar.

### Executor
Decide como e onde as tasks são executadas.

### Webserver
Interface visual para acompanhar e operar o Airflow.

### Metadata Database
Armazena o estado de tudo: DAGs, execuções, variáveis, conexões.

---

## 10) Por que usar Airflow?

Airflow é útil porque resolve problemas comuns de automação e pipelines de dados que scripts soltos ou crontabs não resolvem bem.

### Benefícios principais
- organiza dependências entre tarefas de forma clara
- oferece retries automáticos em caso de falha
- fornece histórico completo de execuções
- centraliza logs de cada etapa
- permite reprocessar (backfill) períodos passados
- dá visibilidade visual sobre pipelines complexos
- integra facilmente com bancos, cloud e APIs via providers

### Exemplos práticos
Airflow é muito usado para:
- pipelines de ETL/ELT
- treinamento periódico de modelos de machine learning
- geração de relatórios agendados
- orquestração de tarefas entre múltiplos sistemas
- automações que dependem de ordem e horário

---

## 11) Quando Airflow faz mais sentido?

Airflow vale a pena quando você precisa de:
- várias tarefas com dependências entre si
- agendamento confiável (não apenas um cron simples)
- visibilidade sobre sucesso/falha de cada etapa
- retries e alertas automáticos
- histórico de execuções para auditoria

### Casos comuns
- pipeline: extrair → transformar → carregar
- pipeline: esperar arquivo → processar → notificar
- pipeline: treinar modelo → validar → publicar
- orquestração de múltiplos jobs de dados que dependem uns dos outros

### Quando pode ser exagero
- uma única tarefa simples, sem dependências, sem necessidade de retries sofisticados — aí um cron job simples pode bastar

---

## 12) O que Airflow não é

Alguns pontos importantes para não confundir:

- Airflow **não processa dados** ele mesmo (não é um motor de processamento como Spark) — ele **orquestra** chamadas para outras ferramentas
- Airflow **não é uma ferramenta de streaming** — ele é voltado para workflows agendados/batch
- Airflow **não substitui** boas práticas de engenharia de dados, apenas organiza a execução

---

## 13) Erros comuns de quem está começando

### Achar que Airflow processa os dados
Ele orquestra e dispara processamento, mas o trabalho pesado normalmente é feito por outra ferramenta (Spark, banco de dados, script Python, etc.).

### Confundir DAG com Task
DAG é o workflow inteiro; Task é um passo dentro dele.

### Escrever lógica pesada direto no corpo da DAG
Código que roda toda vez que o Scheduler lê o arquivo deve ser leve — lógica pesada deve ficar dentro das tasks.

### Não entender a diferença entre Scheduler e Executor
Scheduler decide **quando**; Executor decide **como/onde**.

### Ignorar o Metadata Database
Sem ele saudável, o Airflow perde confiabilidade.

---

## 14) Regra mental para começar bem

Pense em Airflow assim:

- **DAG** → o workflow
- **Task** → um passo do workflow
- **Operator** → o tipo de passo
- **Scheduler** → decide quando roda
- **Executor** → decide como/onde roda
- **Webserver** → onde você acompanha tudo
- **Metadata DB** → onde tudo fica registrado

Se essa lógica estiver clara, os próximos tópicos ficam muito mais fáceis.

---

## Resumo do tópico

Neste ponto, o essencial é entender que Airflow serve para **orquestrar workflows como código, com agendamento, dependências e visibilidade completa sobre execução**.

A base conceitual é:

- **DAG** = workflow
- **Task** = passo do workflow
- **Operator** = tipo de tarefa
- **Scheduler** = quando roda
- **Executor** = como/onde roda
- **Webserver** = visibilidade e operação
- **Metadata DB** = estado e histórico

Com isso claro, o próximo passo natural é aprender a **instalar e configurar o ambiente Airflow corretamente**.
