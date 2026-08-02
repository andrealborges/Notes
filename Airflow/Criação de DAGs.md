# Airflow — Criação de DAGs

Este tópico sai da operação de exemplos prontos e entra na criação dos seus próprios workflows. O objetivo é responder:
> "Como eu escrevo uma DAG do zero, defino tasks, dependências e faço elas trocarem informação entre si?"

Aqui estão a estrutura básica de uma DAG, os principais Operators, formas de definir dependências, agendamento e boas práticas para escrever DAGs sustentáveis.

---

## 1) Anatomia básica de uma DAG

Uma DAG é, no fim das contas, um arquivo Python dentro da pasta `dags/`.

```python
from airflow import DAG
from airflow.operators.empty import EmptyOperator
from datetime import datetime

with DAG(
    dag_id="minha_primeira_dag",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
) as dag:

    inicio = EmptyOperator(task_id="inicio")
    fim = EmptyOperator(task_id="fim")

    inicio >> fim
```

### Elementos principais
- `dag_id` → identificador único da DAG
- `start_date` → a partir de quando a DAG passa a existir para o Scheduler
- `schedule` → frequência de execução
- `catchup` → se o Airflow deve rodar automaticamente todas as execuções "perdidas" desde o `start_date`

---

## 2) `schedule`: definindo a frequência

O parâmetro `schedule` aceita:

### Presets prontos
```python
schedule="@daily"
schedule="@hourly"
schedule="@weekly"
schedule="@once"
```

### Expressão cron
```python
schedule="0 6 * * *"   # todo dia às 6h
```

### Sem agendamento automático
```python
schedule=None
```

### Regra prática
- **`schedule=None`** → a DAG só roda quando disparada manualmente
- **cron/preset** → o Scheduler dispara automaticamente

---

## 3) `catchup`: cuidado com esse parâmetro

Se `catchup=True` (padrão histórico do Airflow), o Airflow tenta rodar **todas** as execuções entre `start_date` e agora, não apenas a mais recente.

### Exemplo do problema
Se `start_date` for de 6 meses atrás e o `schedule` for diário, o Airflow pode tentar disparar centenas de execuções de uma vez.

### Regra prática
Em boa parte dos casos do dia a dia, especialmente em ambientes de estudo, prefira:

```python
catchup=False
```

E use `backfill` manual quando realmente precisar reprocessar o passado.

---

## 4) Criando tasks com PythonOperator

```python
from airflow.operators.python import PythonOperator

def extrair_dados():
    print("extraindo dados...")

extrair = PythonOperator(
    task_id="extrair_dados",
    python_callable=extrair_dados,
)
```

### Quando usar
Sempre que a lógica da task for escrita em Python — é o Operator mais usado no dia a dia.

---

## 5) Criando tasks com BashOperator

```python
from airflow.operators.bash import BashOperator

rodar_script = BashOperator(
    task_id="rodar_script",
    bash_command="python /scripts/processa.py",
)
```

### Quando usar
Quando a task chama um comando de shell, script externo ou ferramenta de linha de comando.

---

## 6) EmptyOperator: organizando o fluxo

```python
from airflow.operators.empty import EmptyOperator

inicio = EmptyOperator(task_id="inicio")
```

Não executa nenhuma lógica — serve apenas como marcador visual/estrutural, útil para agrupar dependências (ex.: várias tasks convergindo em um ponto único).

---

## 7) Definindo dependências entre tasks

A forma mais comum é usar os operadores `>>` e `<<`.

```python
extrair >> transformar >> carregar
```

Isso significa: `extrair` roda primeiro, depois `transformar`, depois `carregar`.

### Dependências em paralelo

```python
extrair >> [transformar_a, transformar_b] >> carregar
```

Nesse caso, `transformar_a` e `transformar_b` rodam em paralelo após `extrair`, e `carregar` só roda depois que ambas terminarem.

### Forma alternativa (métodos explícitos)

```python
transformar.set_upstream(extrair)
carregar.set_upstream(transformar)
```

### Regra prática
- `>>` → "depois de"
- `<<` → "antes de"
- ambos os formatos fazem a mesma coisa; `>>` costuma ser mais lido e usado

---

## 8) Sensors: esperando uma condição

Sensors são um tipo especial de Operator que fica **esperando** até que uma condição seja satisfeita.

```python
from airflow.sensors.filesystem import FileSensor

esperar_arquivo = FileSensor(
    task_id="esperar_arquivo",
    filepath="/dados/entrada.csv",
    poke_interval=30,
    timeout=600,
)
```

### Exemplos de uso comum
- esperar um arquivo aparecer
- esperar outra DAG terminar (`ExternalTaskSensor`)
- esperar um horário específico
- esperar uma condição customizada (`PythonSensor`)

### Regra prática
Sensors consomem um slot de execução enquanto esperam — em uso intenso, vale considerar o modo `reschedule` em vez do modo padrão `poke`, para liberar recursos entre as verificações.

---

## 9) XComs: troca de dados entre tasks

**XCom** (*cross-communication*) é o mecanismo do Airflow para uma task passar um pequeno valor para outra.

```python
def extrair(**context):
    valor = 42
    context["ti"].xcom_push(key="quantidade", value=valor)

def transformar(**context):
    valor = context["ti"].xcom_pull(key="quantidade", task_ids="extrair_dados")
    print(f"valor recebido: {valor}")
```

Com o **TaskFlow API** (mais moderno), isso fica mais simples:

```python
from airflow.decorators import dag, task
from datetime import datetime

@dag(start_date=datetime(2026, 1, 1), schedule="@daily", catchup=False)
def pipeline_exemplo():

    @task
    def extrair():
        return 42

    @task
    def transformar(valor):
        print(f"valor recebido: {valor}")

    transformar(extrair())

pipeline_exemplo()
```

### Regra prática
- XCom é feito para **pequenos valores** (IDs, contagens, caminhos de arquivo)
- **não** é indicado para trafegar volumes grandes de dados (DataFrames inteiros, por exemplo) — para isso, use armazenamento externo (S3, banco, disco) e passe apenas a referência via XCom

---

## 10) TaskFlow API vs. Operators tradicionais

O Airflow moderno oferece a **TaskFlow API** (decorators `@dag` e `@task`), que reduz boilerplate para tasks Python.

### Regra prática
- **lógica em Python, com troca de dados entre tasks** → TaskFlow API costuma deixar o código mais limpo
- **Bash, Sensors, Operators de providers (banco, cloud, etc.)** → continuam usando a forma tradicional de Operators

Os dois estilos podem ser combinados na mesma DAG.

---

## 11) Retries e tratamento de falhas

Cada task pode ter sua própria política de retry:

```python
from datetime import timedelta

extrair = PythonOperator(
    task_id="extrair_dados",
    python_callable=extrair_dados,
    retries=3,
    retry_delay=timedelta(minutes=5),
)
```

### O que isso faz
Se a task falhar, o Airflow tenta novamente até 3 vezes, esperando 5 minutos entre as tentativas.

### Regra prática
Tasks que dependem de sistemas externos instáveis (APIs, redes) se beneficiam muito de `retries` bem configurados.

---

## 12) Argumentos padrão (`default_args`)

Em vez de repetir parâmetros em cada task, é comum definir um `default_args` para a DAG inteira:

```python
default_args = {
    "retries": 2,
    "retry_delay": timedelta(minutes=5),
    "owner": "time-dados",
}

with DAG(
    dag_id="pipeline_exemplo",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
    default_args=default_args,
) as dag:
    ...
```

Esses valores são aplicados a todas as tasks da DAG, a menos que sejam sobrescritos individualmente.

---

## 13) Idempotência: princípio essencial

Uma DAG bem escrita deve ser **idempotente**: rodar a mesma task duas vezes para o mesmo período deve produzir o mesmo resultado, sem duplicar dados ou causar efeitos colaterais indesejados.

### Por que isso importa
- retries automáticos podem executar a task mais de uma vez
- backfill pode reprocessar o mesmo período
- reexecuções manuais são comuns durante depuração

### Regra prática
Ao escrever a lógica de uma task, sempre se pergunte: "o que acontece se essa task rodar duas vezes para a mesma data?"

---

## 14) Evitando lógica pesada no nível da DAG

O corpo do arquivo Python da DAG é lido periodicamente pelo Scheduler — não apenas quando a DAG executa.

### Errado (lógica pesada fora de uma task)

```python
dados = requests.get("https://api.exemplo.com/dados").json()  # roda a cada leitura do arquivo!
```

### Certo (lógica dentro de uma task)

```python
@task
def buscar_dados():
    return requests.get("https://api.exemplo.com/dados").json()
```

### Regra prática
O corpo da DAG deve conter apenas a **definição** da estrutura (tasks e dependências), nunca processamento pesado ou chamadas de rede diretas.

---

## 15) Exemplo real: pipeline simples de ETL

```python
from airflow.decorators import dag, task
from datetime import datetime, timedelta

default_args = {
    "retries": 2,
    "retry_delay": timedelta(minutes=5),
}

@dag(
    dag_id="etl_vendas",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
    default_args=default_args,
)
def etl_vendas():

    @task
    def extrair():
        return {"linhas": 100}

    @task
    def transformar(dados):
        dados["linhas_validas"] = dados["linhas"] - 5
        return dados

    @task
    def carregar(dados):
        print(f"carregando {dados['linhas_validas']} linhas no banco")

    carregar(transformar(extrair()))

etl_vendas()
```

### O que essa DAG faz
- extrai dados (simulado)
- transforma/valida
- carrega no destino final
- roda todo dia, sem tentar reprocessar automaticamente o passado (`catchup=False`)
- tenta novamente até 2 vezes em caso de falha

---

## 16) Erros comuns de quem está começando a escrever DAGs

### Esquecer `catchup=False` e disparar dezenas de execuções acidentalmente
Revise sempre `start_date` e `catchup` juntos.

### Colocar chamadas de API ou banco direto no corpo da DAG
Isso roda a cada varredura do Scheduler, não apenas na execução — pode gerar chamadas excessivas e lentidão.

### Usar XCom para trafegar dados grandes
XCom é para valores pequenos; dados grandes devem ficar em armazenamento externo.

### Esquecer de tratar idempotência
Reexecuções (retry, backfill, clear) podem duplicar efeitos se a task não for idempotente.

### Task_id duplicado dentro da mesma DAG
Cada task precisa de um `task_id` único dentro da DAG.

### Não definir dependências corretamente
Sem `>>`/`<<` (ou chamadas encadeadas na TaskFlow API), as tasks rodam sem ordem garantida entre si.

---

## 17) Regra mental para memorizar

Pense assim ao escrever uma DAG:

- **estrutura da DAG** → só define tasks e dependências, nada de lógica pesada
- **Operator/`@task`** → onde a lógica de verdade acontece
- **`>>` / `<<`** → define a ordem
- **XCom** → troca pequenos valores entre tasks
- **retries + idempotência** → tornam a DAG resiliente a falhas e reexecuções
- **`catchup=False`** → padrão seguro para a maioria dos casos de estudo/desenvolvimento

---

## Resumo do tópico

Neste ponto, o essencial é saber estruturar uma DAG do zero:

- definir `dag_id`, `start_date`, `schedule` e `catchup`
- criar tasks com Operators (`PythonOperator`, `BashOperator`, TaskFlow `@task`)
- conectar tasks com `>>`/`<<`
- trocar pequenos dados entre tasks via XCom
- configurar retries e pensar em idempotência

Com isso, você já consegue escrever workflows próprios e funcionais. O próximo passo natural é explorar **conceitos mais avançados**, como Connections, Variables, Hooks, Task Groups e alertas.
