# Airflow — Instalação e configuração do ambiente

Este tópico sai da teoria e entra na preparação do ambiente. O objetivo é responder:
> "O que eu preciso instalar e configurar para começar a usar Airflow, tanto de forma local (pip) quanto via Docker?"

Aqui estão os dois caminhos mais comuns para rodar Airflow: instalação "normal" com Python/venv, e instalação via Docker Compose (o jeito recomendado pela própria documentação oficial para ambientes de estudo e desenvolvimento).

---

## 1) Duas formas principais de instalar

- **Instalação local (pip + venv)** → mais direta para estudar o funcionamento interno, mas exige mais configuração manual
- **Docker / Docker Compose** → mais próxima de um ambiente real, isola dependências, é o caminho recomendado pela documentação oficial para começar rápido

### Regra prática
- **quer entender o Airflow por dentro, sem muitas dependências extras** → instalação local
- **quer subir um ambiente completo e realista rapidamente** → Docker Compose

---

## 2) Instalação local: pré-requisitos

Antes de instalar, é importante confirmar a versão do Python compatível com a versão do Airflow escolhida (cada versão do Airflow suporta um range específico de versões do Python).

### Recomendado
- usar um ambiente virtual (`venv`) isolado por projeto
- nunca instalar o Airflow direto no Python global da máquina

---

## 3) Instalação local: criando o ambiente virtual

```bash
python -m venv .venv
```

Ativando o ambiente:

```bash
# Linux / macOS / WSL
source .venv/bin/activate
```

```powershell
# Windows (PowerShell)
.venv\Scripts\Activate.ps1
```

---

## 4) Instalação local: definindo `AIRFLOW_HOME`

O Airflow usa uma variável de ambiente chamada `AIRFLOW_HOME` para saber onde guardar DAGs, logs, plugins e configuração.

```bash
export AIRFLOW_HOME=~/airflow
```

Se essa variável não for definida, o Airflow usa um caminho padrão (`~/airflow`) automaticamente.

### Regra prática
Defina `AIRFLOW_HOME` de forma explícita para saber exatamente onde os arquivos estão sendo criados.

---

## 5) Instalação local: instalando o Airflow com constraints

A documentação oficial recomenda instalar usando um **arquivo de constraints**, que trava as versões das dependências para evitar conflitos.

```bash
AIRFLOW_VERSION=2.9.0
PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"

pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"
```

### Por que usar constraints?
Porque o Airflow tem muitas dependências, e instalar sem constraints pode gerar conflitos difíceis de depurar.

---

## 6) Instalação local: inicializando o banco de metadados

Antes do primeiro uso, é preciso inicializar o banco onde o Airflow guarda seu estado:

```bash
airflow db migrate
```

Em versões mais antigas, esse comando era `airflow db init`.

---

## 7) Instalação local: criando o usuário administrador

Para acessar a interface web, é necessário um usuário:

```bash
airflow users create \
  --username admin \
  --firstname Admin \
  --lastname User \
  --role Admin \
  --email admin@example.com \
  --password admin
```

---

## 8) Instalação local: subindo o Webserver e o Scheduler

O Airflow precisa de pelo menos dois processos rodando:

```bash
airflow webserver --port 8080
```

E em outro terminal:

```bash
airflow scheduler
```

### Regra prática
- **Webserver** → interface visual
- **Scheduler** → dispara as execuções

Sem os dois processos ativos, o ambiente não funciona de verdade.

---

## 9) Instalação local: primeiro acesso

Depois de subir os processos, acesse:

```text
http://localhost:8080
```

E entre com o usuário criado no passo 7.

---

## 10) Docker: por que é o caminho recomendado para começar

Rodar Airflow via Docker evita boa parte da dor de configurar Python, dependências e serviços auxiliares (como o banco de metadados) manualmente.

A própria documentação oficial disponibiliza um `docker-compose.yaml` de referência, pronto para uso em ambientes de estudo/desenvolvimento.

---

## 11) Docker: baixando o `docker-compose.yaml` oficial

```bash
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.9.0/docker-compose.yaml'
```

Esse arquivo já vem com:
- Webserver
- Scheduler
- banco de metadados (Postgres)
- Redis (para o CeleryExecutor)
- workers

---

## 12) Docker: criando as pastas necessárias

O Compose oficial espera algumas pastas na raiz do projeto:

```bash
mkdir -p ./dags ./logs ./plugins ./config
```

### Para que servem
- `dags/` → onde ficam os arquivos Python das suas DAGs
- `logs/` → logs de execução das tasks
- `plugins/` → plugins customizados do Airflow
- `config/` → configurações adicionais

---

## 13) Docker: configurando o usuário (Linux)

Em ambientes Linux, é comum definir o UID do usuário para evitar problemas de permissão nos volumes:

```bash
echo -e "AIRFLOW_UID=$(id -u)" > .env
```

No Windows/WSL, esse passo costuma ser menos crítico, mas vale manter o `.env` por consistência com a documentação oficial.

---

## 14) Docker: inicializando o banco de metadados

Antes do primeiro `up`, é necessário rodar a inicialização:

```bash
docker compose up airflow-init
```

Esse comando:
- cria as tabelas do banco de metadados
- cria o usuário administrador padrão (`airflow` / `airflow`, no exemplo oficial)

---

## 15) Docker: subindo o ambiente completo

```bash
docker compose up -d
```

Isso sobe todos os serviços definidos no Compose: webserver, scheduler, banco, redis e workers.

### Ver status dos serviços

```bash
docker compose ps
```

---

## 16) Docker: primeiro acesso

Depois que os serviços estiverem no ar (pode levar um tempo até o Webserver responder), acesse:

```text
http://localhost:8080
```

Usuário e senha padrão do Compose oficial, salvo customização:

```text
usuário: airflow
senha: airflow
```

---

## 17) Docker: adicionando suas próprias DAGs

Basta colocar o arquivo `.py` da DAG dentro da pasta `dags/` local.

Como essa pasta é montada como volume dentro dos containers, o Airflow detecta o novo arquivo automaticamente (pode levar alguns segundos, de acordo com o intervalo de varredura do Scheduler).

---

## 18) Docker: derrubando o ambiente

```bash
docker compose down
```

Para remover também os volumes (banco de metadados, etc.):

```bash
docker compose down --volumes
```

### Atenção
Remover os volumes apaga o histórico de execuções e DAGs registradas no banco.

---

## 19) Checklist antes de seguir em frente

Antes de partir para o dia a dia do Airflow, vale confirmar:

- Webserver acessível em `http://localhost:8080`
- Scheduler rodando (na UI, DAGs aparecem sem erro de importação)
- pasta `dags/` mapeada corretamente
- usuário administrador funcionando
- (Docker) `docker compose ps` mostrando os serviços saudáveis

---

## 20) Erros comuns de instalação e configuração

### Esquecer de rodar `airflow db migrate` (instalação local)
Sem isso, o Airflow não tem onde registrar DAGs e execuções.

### Não definir `AIRFLOW_HOME`
O Airflow cria os arquivos em um local padrão que pode não ser o esperado.

### Subir o Compose sem rodar `airflow-init` antes
O banco de metadados pode não estar pronto e a UI falha ao carregar.

### Esquecer de criar as pastas `dags/`, `logs/`, `plugins/`
O Compose oficial espera essa estrutura; sem ela, os volumes podem ser montados de forma inconsistente.

### Confundir problema de permissão de volume no Linux
Não configurar `AIRFLOW_UID` pode gerar arquivos de log com dono incorreto.

### Achar que o Scheduler detecta a DAG instantaneamente
Existe um intervalo de varredura; a DAG pode levar alguns segundos para aparecer na UI.

---

## 21) Regra mental para este tópico

Pense na instalação e configuração assim:

- instalação local → mais controle, mais configuração manual
- Docker Compose → mais rápido, mais próximo de um ambiente real
- sempre inicializar o banco de metadados antes do primeiro uso
- sempre garantir que Webserver **e** Scheduler estejam rodando
- pasta `dags/` é o ponto de entrada para suas próprias DAGs

---

## Resumo do tópico

Neste ponto, o essencial é deixar o ambiente Airflow funcional e validado, seja localmente ou via Docker.

A base prática é:

- instalação local: `venv` → `AIRFLOW_HOME` → `pip install` com constraints → `airflow db migrate` → criar usuário → subir `webserver` e `scheduler`
- Docker: baixar `docker-compose.yaml` oficial → criar pastas → `airflow-init` → `docker compose up -d`

Com isso pronto, o próximo passo natural é aprender os **comandos e fluxos essenciais do dia a dia com Airflow**.
