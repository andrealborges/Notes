# Airflow — Conceitos avançados

Este tópico sai da criação básica de DAGs e entra em recursos que aparecem quando o uso do Airflow amadurece. O objetivo é responder:
> "Depois que eu já sei escrever uma DAG simples, o que mais eu preciso conhecer para organizar pipelines maiores, integrar com sistemas externos e receber alertas?"

Aqui estão Task Groups, Connections, Variables, Hooks, Providers, SLA/alertas e o motivo de evitar SubDAGs.

---

## 1) Task Groups: organizando DAGs grandes

Quando uma DAG cresce e tem muitas tasks relacionadas, agrupá-las visualmente ajuda bastante.

```python
from airflow.utils.task_group import TaskGroup

with TaskGroup("processamento") as processamento:
    transformar_a = PythonOperator(task_id="transformar_a", python_callable=transformar_a_fn)
    transformar_b = PythonOperator(task_id="transformar_b", python_callable=transformar_b_fn)

extrair >> processamento >> carregar
```

### O que isso faz
Na UI (Graph View), as tasks `transformar_a` e `transformar_b` aparecem agrupadas visualmente sob `processamento`, facilitando a leitura de DAGs grandes.

### Regra prática
Task Group é apenas organização visual/lógica — não é um mecanismo de isolamento de execução.

---

## 2) Por que evitar SubDAGs

Em versões mais antigas do Airflow, existia o conceito de **SubDAG** (uma DAG dentro de outra). Hoje, esse padrão é **desencorajado** pela própria documentação oficial.

### Problemas conhecidos
- pode travar o Scheduler em cenários de alta concorrência
- dificulta debugging
- foi majoritariamente substituído por **Task Groups**

### Regra prática
Para organizar tasks relacionadas → use **Task Groups**.
Para reaproveitar lógica entre DAGs diferentes → considere **DAGs independentes conectadas por `ExternalTaskSensor` ou `TriggerDagRunOperator`**, não SubDAGs.

---

## 3) Connections: configurando acesso a sistemas externos

Uma **Connection** guarda as credenciais e parâmetros de acesso a um sistema externo (banco, API, cloud), configurada na UI (**Admin → Connections**) ou via variável de ambiente/CLI.

### Exemplo de uso em código

```python
from airflow.providers.postgres.hooks.postgres import PostgresHook

hook = PostgresHook(postgres_conn_id="meu_postgres")
registros = hook.get_records("SELECT * FROM vendas LIMIT 10")
```

### Regra prática
Nunca hardcode usuário/senha dentro da DAG — sempre use uma Connection e referencie pelo `conn_id`.

---

## 4) Variables: parametrizando DAGs

**Variables** são pares chave/valor globais, configuráveis na UI (**Admin → Variables**) ou via CLI.

```python
from airflow.models import Variable

limite = Variable.get("limite_linhas_processamento", default_var=1000)
```

### Quando usar
- valores que mudam sem precisar alterar código (limites, flags, caminhos)
- configuração compartilhada entre múltiplas DAGs

### Atenção
Acessar Variables/Connections diretamente no **corpo da DAG** (fora de uma task) gera uma consulta ao banco de metadados a cada varredura do Scheduler. Prefira acessar dentro das tasks sempre que possível.

---

## 5) Hooks: a ponte para sistemas externos

Um **Hook** é uma interface de baixo nível para se conectar a um sistema externo, usando uma Connection configurada.

### Exemplos comuns
- `PostgresHook` → conexão com PostgreSQL
- `S3Hook` → integração com Amazon S3
- `HttpHook` → chamadas HTTP genéricas

### Regra prática
- **Operator** → executa uma ação completa (ex.: `PostgresOperator` roda uma query)
- **Hook** → dá acesso programático ao sistema, usado dentro de uma `PythonOperator`/`@task` quando você precisa de mais controle do que um Operator pronto oferece

---

## 6) Providers: estendendo o Airflow

**Providers** são pacotes separados (`apache-airflow-providers-*`) que adicionam Operators, Hooks e Sensors para integrações específicas.

### Exemplos

```bash
pip install apache-airflow-providers-amazon
pip install apache-airflow-providers-postgres
pip install apache-airflow-providers-slack
```

### Regra prática
O "core" do Airflow é enxuto de propósito — praticamente toda integração com sistema externo (cloud, banco, mensageria) vem de um provider específico, instalado à parte.

---

## 7) TriggerDagRunOperator: uma DAG disparando outra

```python
from airflow.operators.trigger_dagrun import TriggerDagRunOperator

disparar_outra = TriggerDagRunOperator(
    task_id="disparar_outra_dag",
    trigger_dag_id="dag_dependente",
)
```

### Quando usar
Quando o fim de uma DAG deve iniciar outra automaticamente, mantendo-as como unidades independentes (diferente do padrão de SubDAG).

---

## 8) ExternalTaskSensor: esperando outra DAG terminar

```python
from airflow.sensors.external_task import ExternalTaskSensor

esperar_outra_dag = ExternalTaskSensor(
    task_id="esperar_outra_dag",
    external_dag_id="dag_de_origem",
    external_task_id="task_final",
)
```

### Quando usar
Quando uma DAG precisa esperar que **outra DAG** (não apenas outra task da mesma DAG) termine antes de continuar.

---

## 9) SLA: alertando sobre atrasos

O Airflow permite definir um **SLA** (Service Level Agreement) por task — um tempo máximo esperado de execução.

```python
from datetime import timedelta

processar = PythonOperator(
    task_id="processar",
    python_callable=processar_fn,
    sla=timedelta(minutes=30),
)
```

Se a task ultrapassar esse tempo, o Airflow registra uma violação de SLA e pode disparar notificações configuradas.

---

## 10) Alertas em caso de falha

É possível configurar callbacks para notificar automaticamente quando uma task falha:

```python
def notificar_falha(context):
    print(f"Task falhou: {context['task_instance'].task_id}")

processar = PythonOperator(
    task_id="processar",
    python_callable=processar_fn,
    on_failure_callback=notificar_falha,
)
```

### Uso comum em produção
Esses callbacks costumam ser usados para enviar alertas via e-mail, Slack ou outra ferramenta de observabilidade, normalmente usando um provider específico (ex.: `apache-airflow-providers-slack`).

---

## 11) Pools: controlando concorrência

Um **Pool** limita quantas tasks podem rodar simultaneamente em um determinado recurso.

```python
consultar_api = PythonOperator(
    task_id="consultar_api",
    python_callable=consultar_api_fn,
    pool="pool_api_externa",
)
```

### Quando usar
Quando um sistema externo tem limite de requisições simultâneas (rate limit), e você quer garantir que o Airflow não sobrecarregue esse sistema mesmo com várias DAGs/tasks concorrentes.

---

## 12) Erros comuns em uso mais avançado

### Usar SubDAG por hábito de tutoriais antigos
A recomendação atual é Task Group ou DAGs independentes conectadas por sensor/trigger.

### Acessar Variable/Connection direto no corpo da DAG
Gera carga desnecessária no banco de metadados a cada leitura do arquivo pelo Scheduler.

### Não configurar Pools para sistemas externos sensíveis a carga
Pode gerar throttling ou bloqueio por parte do sistema externo.

### Ignorar SLA e alertas em pipelines críticos
Sem isso, falhas silenciosas podem passar despercebidas por horas.

### Instalar providers demais "por garantia"
Aumenta a superfície de dependências sem necessidade — instale apenas os providers realmente usados.

---

## 13) Regra mental para memorizar

Pense assim:

- **Task Group** → organiza tasks dentro da mesma DAG
- **TriggerDagRunOperator / ExternalTaskSensor** → conecta DAGs diferentes
- **Connection** → como se conectar a um sistema externo
- **Variable** → configuração parametrizável
- **Hook** → acesso programático a um sistema externo
- **Provider** → pacote que adiciona integrações específicas
- **SLA / callbacks** → alertas sobre atraso ou falha
- **Pool** → controla concorrência sobre um recurso limitado

---

## Resumo do tópico

Neste ponto, o essencial é saber que, além de DAGs e tasks básicas, o Airflow oferece um conjunto de recursos para maturidade operacional:

- organização com **Task Groups**
- integração entre DAGs com **TriggerDagRunOperator** e **ExternalTaskSensor**
- configuração externa via **Connections** e **Variables**
- extensão via **Providers** e **Hooks**
- observabilidade via **SLA** e **callbacks de falha**
- controle de carga via **Pools**

Com essa base, você já tem uma visão completa — do conceito básico até o uso mais maduro — para operar Airflow com confiança em cenários reais.