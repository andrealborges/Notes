# WSL — Docker + WSL (arquitetura, funcionamento e prática)

Este tópico explica **como o Docker realmente funciona quando integrado ao WSL 2**, onde os containers rodam, como portas e volumes são mapeados e quais decisões impactam performance.  
Aqui está a base para entender Postgres, APIs, pipelines e containers no seu ambiente local **sem achismo**.

---

## 1) Por que o Docker Desktop usa WSL 2?

O Docker foi criado para Linux e depende fortemente de recursos do **kernel Linux**, como:

- namespaces
- cgroups
- overlay filesystem
- networking do kernel

No Windows, existem duas opções:
1) **Emular ou reimplementar esses recursos**
2) **Rodar um kernel Linux real**

O Docker Desktop escolheu a **opção 2**, usando o **WSL 2**, porque:
- o WSL 2 já fornece um kernel Linux real
- a VM é leve e integrada ao Windows
- performance é muito melhor do que soluções antigas (Docker Toolbox, Hyper-V VM manual)

**Resumo:**  
> Docker Desktop usa WSL 2 porque é a forma mais próxima de rodar Docker “nativo Linux” dentro do Windows.

---

## 2) O Docker roda dentro do WSL ou fora dele?

Resposta curta: **os containers rodam dentro do Linux do WSL 2**.

Arquitetura simplificada:

```

Windows
├─ Docker Desktop (UI + controle)
├─ WSL 2
│   ├─ Kernel Linux
│   ├─ Docker Engine (dockerd)
│   └─ Containers

````

- O **Docker Desktop** (app Windows) é:
  - interface gráfica
  - gerenciador de configurações
  - controlador do lifecycle do Docker
- O **Docker Engine** (`dockerd`) roda **dentro do WSL**
- Os **containers** são processos Linux rodando nesse kernel

Você pode comprovar isso entrando no WSL e rodando:

```bash
ps aux | grep dockerd
```

---

## 3) Diferença entre Docker no Windows e Docker no WSL

### Docker no Windows (conceito)

* Docker Desktop instalado como aplicativo Windows
* UI, settings, login, extensões
* Integração com WSL

### Docker no WSL (execução real)

* Docker Engine rodando no Linux
* Containers Linux reais
* Volumes, redes, processos Linux

### Na prática

Quando você roda:

```bash
docker ps
```

Não importa se o comando foi:

* no PowerShell
* no terminal WSL
* no VS Code (WSL)

➡️ **os containers são os mesmos**, porque todos falam com o mesmo Docker Engine.

---

## 4) Onde os containers realmente estão rodando?

Eles estão rodando:

* no **kernel Linux do WSL 2**
* como processos Linux
* isolados por namespaces e cgroups

Você pode ver isso pelo WSL:

```bash
ps aux | grep postgres
```

Ou ver o PID do container:

```bash
docker inspect --format '{{.State.Pid}}' nome_container
```

Esse PID existe **no Linux do WSL**, não no Windows.

---

## 5) Onde ficam os volumes do Docker no WSL?

### Volumes nomeados (recomendado)

Quando você usa:

```yaml
volumes:
  pgdata:
```

Ou:

```bash
docker volume create pgdata
```

Esses dados ficam:

* dentro do filesystem Linux do WSL
* normalmente em algo como:

  ```
  /var/lib/docker/volumes/...
  ```

Fisicamente:

* isso está dentro do **VHDX do WSL**
* não é um diretório direto do `C:\`

Você pode inspecionar:

```bash
docker volume inspect pgdata
```

---

### Bind mount (mapeando pasta do Windows)

Exemplo:

```yaml
volumes:
  - ./data:/var/lib/postgresql/data
```

Se `./data` estiver em:

* `/mnt/c/...` → dados no Windows
* `/home/usuario/...` → dados no filesystem Linux

⚠️ **Atenção:**
Bind mounts vindos de `/mnt/c` podem ser **bem mais lentos**.

---

## 6) Como funciona o mapeamento de portas (`5437:5432`)?

Exemplo clássico com Postgres:

```yaml
ports:
  - "5437:5432"
```

Significado:

```
Windows (host)        Container
localhost:5437  -->  5432 (Postgres)
```

Fluxo real:

```
Windows app
 → localhost:5437
 → Docker (WSL)
 → NAT / bridge
 → container:5432
```

### Testando

No Windows:

```powershell
psql -h localhost -p 5437 -U postgres
```

No WSL:

```bash
psql -h localhost -p 5437 -U postgres
```

Ambos funcionam porque o Docker expõe a porta para o host.

---

## 7) O que acontece quando paro o Docker Desktop?

Quando você:

* fecha o Docker Desktop
* ou escolhe “Quit Docker Desktop”

O que acontece:

* Docker Engine (`dockerd`) é parado
* containers são parados
* redes Docker são desmontadas
* volumes **não são apagados**

Depois, ao abrir novamente:

* containers podem ser recriados
* dados persistidos em volumes continuam intactos

⚠️ Importante:

```bash
docker stop
```

≠
Fechar Docker Desktop

Fechar o Docker Desktop **derruba tudo**.

---

## 8) Como acessar um container via WSL?

### Acessar shell de um container

```bash
docker exec -it nome_container bash
```

Ou, se não tiver bash:

```bash
docker exec -it nome_container sh
```

### Acessar banco (Postgres)

```bash
docker exec -it pg_container psql -U postgres
```

### Ver logs

```bash
docker logs -f nome_container
```

### Ver uso de recursos

```bash
docker stats
```

Tudo isso funciona:

* no terminal WSL
* no PowerShell
* no VS Code (WSL)

---

## 9) O que muda em performance ao usar WSL + Docker?

### Pontos fortes

* Containers Linux reais (sem tradução)
* Performance muito próxima de Linux nativo
* Excelente para:

  * Postgres
  * APIs (FastAPI, Flask)
  * Python
  * Spark local
  * pipelines

### Gargalos comuns

* Bind mount em `/mnt/c`
* Muitos arquivos pequenos
* I/O intenso vindo do Windows filesystem

### Boas práticas de performance

* Código e ambientes em:

  ```bash
  /home/usuario/projetos
  ```
* Volumes nomeados para bancos
* Evitar bind mount de `node_modules`, `.venv`, `.cache`

---

## Arquitetura mental correta (resumo visual)

```
Windows
 ├─ VS Code (UI)
 ├─ Docker Desktop (controle)
 └─ PowerShell
        ↓
     WSL 2
     ├─ Kernel Linux
     ├─ Docker Engine
     ├─ Containers
     └─ Volumes (VHDX)
```

---

## Erros comuns (e como evitar)

* ❌ Achar que container roda no Windows

  * ✅ Ele roda no Linux do WSL
* ❌ Salvar dados de banco em `/mnt/c`

  * ✅ Use volumes Docker
* ❌ Mapear muitas pastas do Windows

  * ✅ Trabalhe dentro do `/home`
* ❌ Confundir porta interna com externa

  * ✅ `host:container`