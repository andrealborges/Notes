# Docker — Orquestração local com Docker Compose

Este tópico sai do container isolado e entra em um cenário mais próximo do mundo real. O objetivo é responder:  
> “Como eu subo vários serviços juntos de forma organizada, reproduzível e fácil de manter?”

Aqui estão os conceitos, estrutura e comandos mais importantes para usar **Docker Compose** no dia a dia, conectando aplicação, banco, cache e outros serviços em um único ambiente local.

---

## 1) O que é Docker Compose?

O **Docker Compose** é a forma de definir e subir **múltiplos containers** usando um único arquivo de configuração.

Em vez de criar cada container manualmente com vários `docker run`, você descreve o ambiente em um arquivo YAML, normalmente chamado:

```text
compose.yaml
```

ou

```text
docker-compose.yml
```

### Regra prática

Se você precisa subir:

* aplicação
* banco de dados
* cache
* mensageria
* ferramenta auxiliar

o Compose quase sempre é a forma mais organizada de fazer isso.

---

## 2) Qual problema o Compose resolve?

Sem Compose, um ambiente com vários serviços exige muitos comandos manuais, por exemplo:

* criar rede
* criar volume
* subir banco
* subir API
* publicar portas
* definir variáveis
* garantir ordem de inicialização

Com Compose, isso fica centralizado em um só lugar.

### Benefícios principais

* padroniza o ambiente
* reduz comandos repetitivos
* facilita onboarding
* melhora reprodutibilidade
* simplifica desenvolvimento local
* ajuda a documentar a arquitetura do ambiente

---

## 3) Quando vale a pena usar Compose?

Use Compose quando sua aplicação depende de mais de um serviço.

### Casos típicos

* API + banco PostgreSQL
* frontend + backend
* aplicação + Redis
* aplicação + banco + fila
* laboratório de testes com múltiplos serviços
* stack local de desenvolvimento

### Regra prática

* **um único container simples** → `docker run` pode bastar
* **mais de um serviço** → Compose costuma ser a melhor escolha

---

## 4) Estrutura básica de um arquivo Compose

Exemplo simples:

```yaml
services:
  app:
    image: nginx
    ports:
      - "8080:80"
```

Esse arquivo já define um serviço chamado `app` que usa a imagem `nginx` e publica a porta 80 do container na porta 8080 da máquina.

### Ideia central

No Compose, você descreve:

* serviços
* imagens ou builds
* portas
* volumes
* variáveis
* redes
* dependências

---

## 5) Principais seções do Compose

## `services`

É a parte principal do arquivo.
Cada item em `services` representa um container gerenciado pelo Compose.

Exemplo:

```yaml
services:
  app:
    image: nginx

  db:
    image: postgres:16
```

Nesse caso, o ambiente tem dois serviços:

* `app`
* `db`

---

## `image`

Define a imagem usada pelo serviço.

Exemplo:

```yaml
image: postgres:16
```

Use quando você quer consumir uma imagem já pronta.

---

## `build`

Define que o serviço será construído a partir de um `Dockerfile`.

Exemplo:

```yaml
build: .
```

Ou:

```yaml
build:
  context: .
  dockerfile: Dockerfile
```

### Regra prática

* `image` → usar imagem pronta
* `build` → construir imagem local da sua aplicação

---

## `ports`

Mapeia portas entre host e container.

Exemplo:

```yaml
ports:
  - "8000:8000"
```

### Lógica

```text
HOST:CONTAINER
```

Assim como no `docker run`, esse é um ponto que precisa estar muito claro.

---

## `volumes`

Define persistência ou montagem de arquivos/pastas.

Exemplo:

```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data
```

Ou bind mount:

```yaml
volumes:
  - .:/app
```

### Dois usos comuns

* persistir dados do serviço
* montar o código-fonte local dentro do container

---

## `environment`

Define variáveis de ambiente.

Exemplo:

```yaml
environment:
  APP_ENV: development
  DB_HOST: db
```

Também é comum escrever assim:

```yaml
environment:
  - APP_ENV=development
  - DB_HOST=db
```

---

## `depends_on`

Indica dependência entre serviços.

Exemplo:

```yaml
depends_on:
  - db
```

Isso ajuda a organizar a inicialização, embora não garanta sozinho que o serviço dependente já esteja “pronto para uso” internamente.

---

## 6) Exemplo real: aplicação + PostgreSQL

Aqui está um exemplo prático:

```yaml
services:
  app:
    build: .
    container_name: minha-api
    ports:
      - "8000:8000"
    environment:
      DB_HOST: db
      DB_PORT: 5432
      DB_NAME: appdb
      DB_USER: appuser
      DB_PASSWORD: appsenha
    depends_on:
      - db

  db:
    image: postgres:16
    container_name: meu-postgres
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: appsenha
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### O que esse arquivo faz

* constrói a aplicação localmente
* sobe um PostgreSQL
* conecta os dois serviços na mesma rede do Compose
* persiste os dados do banco em volume nomeado
* expõe a API e o banco para a máquina local

---

## 7) Como os serviços se enxergam

Um dos maiores benefícios do Compose é a rede automática entre os serviços.

No exemplo anterior, o serviço `app` consegue acessar o banco usando:

```text
db
```

como host, porque `db` é o nome do serviço.

### Regra prática

Dentro do Compose:

* containers conversam pelo **nome do serviço**
* normalmente você não usa `localhost` para um container falar com outro

### Exemplo

Se a API quiser acessar o PostgreSQL:

* host: `db`
* porta: `5432`

e não `localhost`

---

## 8) Como subir o ambiente

Para subir todos os serviços:

```bash
docker compose up
```

### Rodar em background

```bash
docker compose up -d
```

Esse é o modo mais comum no dia a dia.

### O que acontece

O Compose:

* lê o arquivo YAML
* cria rede(s)
* cria volume(s), se necessário
* constrói imagem(ns), se houver `build`
* cria e sobe os containers

---

## 9) Como derrubar o ambiente

```bash
docker compose down
```

Esse comando para e remove os containers e a rede criada pelo Compose.

### Atenção

Os volumes nomeados normalmente permanecem, a menos que você peça remoção explícita.

Para remover também os volumes:

```bash
docker compose down -v
```

---

## 10) Como ver logs

### Logs de todos os serviços

```bash
docker compose logs
```

### Acompanhar em tempo real

```bash
docker compose logs -f
```

### Logs de um serviço específico

```bash
docker compose logs app
```

ou

```bash
docker compose logs db
```

Isso facilita muito o diagnóstico em ambientes com vários containers.

---

## 11) Como reconstruir serviços

Se você alterou o código ou o Dockerfile, pode reconstruir e subir de novo:

```bash
docker compose up --build
```

Ou em background:

```bash
docker compose up -d --build
```

### Quando usar

* mudanças no Dockerfile
* mudança em dependências
* necessidade de rebuild explícito

---

## 12) Como listar containers do Compose

```bash
docker compose ps
```

Esse comando mostra os serviços do projeto Compose, com status e portas.

É muito útil porque o foco fica no ambiente atual, sem misturar tudo com containers de outros contextos.

---

## 13) Como executar comandos em um serviço

Para rodar um comando dentro de um serviço em execução:

```bash
docker compose exec app sh
```

Ou:

```bash
docker compose exec db psql -U appuser -d appdb
```

### Quando isso é útil

* abrir shell na aplicação
* testar comando no banco
* verificar arquivos
* depurar ambiente
* validar variáveis

---

## 14) Volumes no Compose

Os volumes são muito importantes em ambientes locais.

### Volume nomeado

```yaml
volumes:
  postgres_data:
```

e no serviço:

```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data
```

Isso permite que os dados sobrevivam à remoção do container.

### Bind mount

```yaml
volumes:
  - .:/app
```

Esse formato monta a pasta local dentro do container.

### Regra prática

* **dados persistentes do serviço** → volume nomeado
* **código-fonte local para desenvolvimento** → bind mount

---

## 15) Variáveis de ambiente e arquivos `.env`

Em muitos projetos, o Compose usa um arquivo `.env` para evitar repetir variáveis sensíveis ou de configuração.

Exemplo:

```env
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=appsenha
```

E no Compose:

```yaml
environment:
  POSTGRES_DB: ${POSTGRES_DB}
  POSTGRES_USER: ${POSTGRES_USER}
  POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
```

### Benefícios

* centraliza configuração
* evita hardcode excessivo
* facilita troca entre ambientes

### Atenção

Mesmo em desenvolvimento, trate dados sensíveis com cuidado.

---

## 16) Ordem de inicialização e limitação do `depends_on`

Esse ponto é importante.

```yaml
depends_on:
  - db
```

Isso indica que o serviço `app` depende do `db`, mas não garante sozinho que o banco já terminou de inicializar internamente e está pronto para conexões.

### Regra prática

`depends_on` ajuda na ordem de subida, mas aplicações mais robustas ainda podem precisar de:

* lógica de retry
* healthchecks
* espera ativa

---

## 17) Exemplo prático com código montado localmente

Para desenvolvimento, é comum montar o código local dentro do container:

```yaml
services:
  app:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - .:/app
    working_dir: /app
```

### Quando isso ajuda

* editar código localmente
* ver mudança refletindo no container
* evitar rebuild a cada pequena alteração, dependendo da stack

---

## 18) Nome do projeto Compose

Por padrão, o Compose agrupa os recursos em um projeto, normalmente baseado no nome da pasta.

Isso influencia nomes de:

* containers
* redes
* volumes

### Exemplo mental

Se a pasta do projeto for `meu-sistema`, o Compose pode criar recursos com prefixos derivados disso.

Isso ajuda a isolar ambientes diferentes no mesmo host.

---

## 19) Erros comuns de quem está começando

### Tentar usar `localhost` entre containers

Dentro do Compose, um serviço normalmente acessa o outro pelo nome do serviço.

### Publicar porta desnecessariamente

Nem todo serviço precisa expor porta para o host.
Às vezes basta que ele esteja acessível apenas para outros containers.

### Esquecer persistência do banco

Sem volume, dados podem ser perdidos ao recriar container.

### Achar que `depends_on` resolve readiness completa

Ele ajuda, mas não substitui verificação real de disponibilidade.

### Misturar muita coisa em um único serviço

O Compose funciona melhor quando cada serviço tem responsabilidade clara.

### Subir ambiente sem entender logs

Logs são uma das ferramentas mais importantes para descobrir falhas de integração.

---

## 20) Fluxo real de uso no dia a dia

### Subir tudo

```bash
docker compose up -d
```

### Ver status

```bash
docker compose ps
```

### Ver logs

```bash
docker compose logs -f
```

### Entrar no serviço da aplicação

```bash
docker compose exec app sh
```

### Rebuildar e subir de novo

```bash
docker compose up -d --build
```

### Derrubar ambiente

```bash
docker compose down
```

Esse fluxo já cobre grande parte do uso local de Compose.

---

## 21) Quando o Compose está sendo bem usado?

Você está usando Compose bem quando:

* consegue subir todo o ambiente com um único comando
* qualquer pessoa do time consegue reproduzir o setup
* banco e aplicação se conectam sem configuração manual improvisada
* volumes persistem o que precisa persistir
* logs ajudam no diagnóstico
* o arquivo YAML está claro e organizado

---

## 22) Regra mental para memorizar

Pense assim:

* `services` → quais containers existem
* `image` / `build` → de onde vem cada serviço
* `ports` → o que fica acessível no host
* `volumes` → o que persiste ou é montado
* `environment` → configuração do runtime
* `depends_on` → dependências entre serviços

E nos comandos:

* `docker compose up` → sobe ambiente
* `docker compose down` → derruba ambiente
* `docker compose logs` → vê logs
* `docker compose ps` → vê status
* `docker compose exec` → executa dentro do serviço