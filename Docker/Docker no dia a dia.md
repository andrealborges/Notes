# Docker — Comandos essenciais do dia a dia

Este tópico sai da instalação e entra no uso real do Docker. O objetivo é responder:  
> “Quais comandos eu realmente preciso saber para trabalhar com containers no dia a dia?”

Aqui estão os comandos mais importantes para baixar imagens, criar containers, inspecionar execução, acessar processos, visualizar logs e remover recursos com segurança.

---

## 1) Como pensar nos comandos do Docker

Antes de decorar comandos, vale entender a lógica:

- **imagem** = modelo pronto
- **container** = instância em execução da imagem

Na prática, o fluxo mais comum é:

1. obter uma imagem  
2. criar e executar um container  
3. inspecionar o que está rodando  
4. acessar logs ou shell  
5. parar, reiniciar ou remover  

Se essa sequência fizer sentido, os comandos ficam muito mais fáceis de memorizar.

---

## 2) Baixando imagens

Para baixar uma imagem de um registry, use:

```bash
docker pull nginx
```

Esse comando baixa a imagem `nginx` para sua máquina.

### Exemplo com tag

```bash
docker pull python:3.12
```

### Regra prática

* sem tag explícita, o Docker normalmente usa `latest`
* para ambientes mais previsíveis, prefira usar uma tag definida

---

## 3) Listando imagens disponíveis

Para ver as imagens já baixadas localmente:

```bash
docker images
```

Esse comando ajuda a responder:

* quais imagens já existem na máquina
* qual o tamanho delas
* qual tag está sendo usada

---

## 4) Criando e executando um container

O comando mais comum para iniciar um container é:

```bash
docker run nginx
```

Isso faz o Docker:

* procurar a imagem localmente
* baixar se necessário
* criar o container
* iniciar o processo principal

Mas no dia a dia, normalmente você usa opções adicionais.

### Exemplo mais realista

```bash
docker run -d -p 8080:80 --name meu-nginx nginx
```

### O que esse comando faz

* `-d` → roda em background
* `-p 8080:80` → publica a porta 80 do container na porta 8080 da máquina
* `--name meu-nginx` → define um nome amigável para o container
* `nginx` → imagem usada

### Regra prática

Esse é um dos formatos mais importantes para memorizar:

```bash
docker run -d -p PORTA_HOST:PORTA_CONTAINER --name NOME IMAGEM
```

---

## 5) Ver containers em execução

Para listar apenas os containers ativos:

```bash
docker ps
```

Esse é um dos comandos mais usados no dia a dia.

Ele mostra:

* ID do container
* imagem
* comando
* tempo de execução
* status
* portas
* nome

---

## 6) Ver todos os containers, inclusive os parados

```bash
docker ps -a
```

Use esse comando quando:

* um container parou inesperadamente
* você quer ver histórico de execução
* precisa remover containers antigos
* quer checar se algo foi criado mas não está mais rodando

---

## 7) Parando um container

Para parar um container em execução:

```bash
docker stop meu-nginx
```

Você pode usar:

* o nome do container
* ou o ID do container

### Exemplo com ID

```bash
docker stop a1b2c3d4e5f6
```

---

## 8) Iniciando novamente um container parado

```bash
docker start meu-nginx
```

Esse comando reinicia um container que já existe, mas está parado.

### Diferença importante

* `docker run` → cria e inicia um novo container
* `docker start` → inicia um container já existente

Essa diferença costuma confundir bastante quem está começando.

---

## 9) Reiniciando um container

```bash
docker restart meu-nginx
```

Útil quando:

* você alterou alguma configuração externa
* quer reiniciar rapidamente um serviço
* precisa reaplicar o estado do processo

---

## 10) Removendo containers

Para remover um container parado:

```bash
docker rm meu-nginx
```

### Atenção

O container normalmente precisa estar parado antes de ser removido.

Se quiser forçar:

```bash
docker rm -f meu-nginx
```

### Regra prática

* `stop` → para
* `rm` → remove
* `rm -f` → força parada e remoção

---

## 11) Removendo imagens

Para remover uma imagem local:

```bash
docker rmi nginx
```

Ou com tag:

```bash
docker rmi python:3.12
```

### Atenção

Se existir container usando essa imagem, a remoção pode falhar até que os containers relacionados sejam removidos.

---

## 12) Ver logs de um container

Esse é um dos comandos mais úteis para diagnóstico:

```bash
docker logs meu-nginx
```

### Acompanhar logs em tempo real

```bash
docker logs -f meu-nginx
```

### Mostrar apenas as últimas linhas

```bash
docker logs --tail 50 meu-nginx
```

### Regra prática

Use `logs` quando quiser descobrir:

* por que o container parou
* se a aplicação iniciou corretamente
* qual erro ocorreu no processo principal
* se o serviço está respondendo normalmente

---

## 13) Executando comandos dentro do container

Para rodar um comando dentro de um container em execução:

```bash
docker exec meu-nginx ls
```

Mas o mais comum é abrir um shell interativo.

### Exemplo com bash

```bash
docker exec -it meu-nginx bash
```

### Exemplo com sh

```bash
docker exec -it meu-nginx sh
```

### O que significa

* `exec` → executa um comando em um container já rodando
* `-i` → mantém entrada interativa
* `-t` → aloca terminal

### Quando usar

* inspecionar arquivos
* testar comandos
* verificar variáveis
* validar diretórios
* fazer troubleshooting rápido

---

## 14) Ver detalhes técnicos de um container

```bash
docker inspect meu-nginx
```

Esse comando retorna detalhes completos em JSON, como:

* IP do container
* volumes montados
* portas
* variáveis de ambiente
* configuração de rede
* caminho do processo principal

É muito útil para diagnóstico, mas tende a ser mais verboso.

---

## 15) Ver uso de recursos

Para acompanhar uso de CPU, memória e rede em tempo real:

```bash
docker stats
```

Ou para um container específico:

```bash
docker stats meu-nginx
```

Esse comando é útil para:

* observar consumo excessivo
* testar comportamento de carga
* identificar gargalos simples

---

## 16) Nomeando containers corretamente

Sempre que possível, use `--name`.

### Exemplo

```bash
docker run -d -p 5432:5432 --name meu-postgres postgres
```

### Por que isso ajuda?

Porque depois fica muito mais fácil usar:

* `docker stop meu-postgres`
* `docker logs meu-postgres`
* `docker exec -it meu-postgres bash`

Em vez de depender de IDs aleatórios.

---

## 17) Publicando portas corretamente

Um dos pontos mais importantes no `docker run` é o mapeamento de portas:

```bash
docker run -p 8080:80 nginx
```

Significa:

* porta `8080` na máquina local
* apontando para a porta `80` dentro do container

### Regra prática

```bash
HOST:CONTAINER
```

### Erro comum

Confundir a porta da aplicação dentro do container com a porta publicada na máquina.

---

## 18) Trabalhando com containers em background

Muitas vezes você não quer “prender” o terminal.

Para isso:

```bash
docker run -d nginx
```

O `-d` significa **detached mode**, ou seja, o container roda em segundo plano.

Depois você usa:

```bash
docker ps
docker logs
docker stop
```

para interagir com ele.

---

## 19) Exemplo de fluxo completo no dia a dia

Aqui está um fluxo simples e muito comum:

### Baixar a imagem

```bash
docker pull nginx
```

### Criar e subir o container

```bash
docker run -d -p 8080:80 --name meu-nginx nginx
```

### Ver se está rodando

```bash
docker ps
```

### Ver logs

```bash
docker logs meu-nginx
```

### Parar o container

```bash
docker stop meu-nginx
```

### Iniciar novamente

```bash
docker start meu-nginx
```

### Remover o container

```bash
docker rm -f meu-nginx
```

Esse já é um ciclo real de uso para muitos testes locais.

---

## 20) Comandos que você mais vai usar de verdade

Se tivesse que priorizar os mais importantes, seriam estes:

```bash
docker pull
docker run
docker ps
docker ps -a
docker stop
docker start
docker restart
docker rm
docker images
docker rmi
docker logs
docker exec
docker inspect
docker stats
```

Esses formam o núcleo do uso operacional do Docker.

---

## 21) Erros comuns de quem está começando

### Confundir `run` com `start`

* `run` cria um novo container
* `start` reutiliza um container já existente

### Esquecer o `-d`

Sem `-d`, o terminal fica preso ao processo principal do container.

### Não nomear container

Isso dificulta logs, stop, exec e manutenção.

### Confundir as portas

Sempre pense em:

```bash
PORTA_HOST:PORTA_CONTAINER
```

### Tentar remover container rodando sem força

Use `docker stop` antes, ou `docker rm -f` se necessário.

### Achar que `exec` inicia container parado

`docker exec` só funciona em container que já está em execução.

---

## 22) Regra mental para memorizar

Pense assim:

* **pull** → baixa imagem
* **images** → lista imagens
* **run** → cria e inicia
* **ps** → lista execução
* **stop/start/restart** → controla ciclo de vida
* **logs** → vê saída da aplicação
* **exec** → entra no container
* **rm/rmi** → remove recursos
* **inspect** → vê detalhes
* **stats** → acompanha consumo

Essa lógica já cobre a maior parte do dia a dia.

---

## Resumo do tópico

Neste ponto, o essencial é dominar o ciclo básico de trabalho com containers:

* baixar imagem
* criar container
* listar execução
* acessar logs
* executar comandos internos
* parar, reiniciar e remover

A base prática é:

* `docker pull`
* `docker run`
* `docker ps`
* `docker stop`
* `docker start`
* `docker logs`
* `docker exec`
* `docker rm`

Com isso, você já consegue operar containers no dia a dia com segurança e entendimento. O próximo passo natural é aprender a **criar suas próprias imagens com Dockerfile**, que é quando você deixa de apenas consumir imagens prontas e passa a empacotar suas próprias aplicações.