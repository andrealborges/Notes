# Docker — Criação de imagens e Dockerfile

Este tópico sai do uso de imagens prontas e entra na construção do seu próprio ambiente. O objetivo é responder:  
> “Como eu transformo minha aplicação em uma imagem Docker reproduzível, organizada e pronta para execução?”

Aqui estão os conceitos, instruções e boas práticas mais importantes para criar imagens com `Dockerfile`, controlar dependências e empacotar aplicações de forma previsível.

---

## 1) O que é um Dockerfile?

O **Dockerfile** é um arquivo de texto com instruções que dizem ao Docker como construir uma imagem.

Ele funciona como uma receita de montagem do ambiente:
- qual imagem base usar
- qual diretório de trabalho definir
- quais arquivos copiar
- quais dependências instalar
- qual comando executar ao iniciar o container

### Regra prática
- **Dockerfile** = receita
- **imagem** = resultado da receita
- **container** = execução da imagem

---

## 2) Por que criar sua própria imagem?

Usar imagens prontas é ótimo para testes rápidos, mas no dia a dia você normalmente precisa empacotar sua aplicação com:
- código-fonte
- dependências
- runtime
- variáveis esperadas
- comando de inicialização

Criar sua própria imagem permite:
- padronizar o ambiente
- evitar “funciona só na minha máquina”
- facilitar deploy
- simplificar onboarding
- reproduzir o mesmo setup em qualquer lugar

---

## 3) Estrutura mínima de um Dockerfile

Um exemplo simples:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY . /app

RUN pip install -r requirements.txt

CMD ["python", "app.py"]
```

### O que esse exemplo faz

* `FROM python:3.12-slim` → define a imagem base
* `WORKDIR /app` → define o diretório de trabalho
* `COPY . /app` → copia os arquivos do projeto
* `RUN pip install -r requirements.txt` → instala dependências
* `CMD ["python", "app.py"]` → define o comando padrão de inicialização

Esse já é o esqueleto básico de muitos projetos.

---

## 4) Principais instruções do Dockerfile

## `FROM`

Define a imagem base da construção.

Exemplo:

```dockerfile
FROM node:20-alpine
```

Essa costuma ser a primeira instrução relevante do arquivo.

### Regra prática

Escolha uma base:

* confiável
* compatível com sua stack
* preferencialmente enxuta quando fizer sentido

---

## `WORKDIR`

Define o diretório de trabalho dentro da imagem.

Exemplo:

```dockerfile
WORKDIR /app
```

Depois disso, comandos como `COPY`, `RUN` e `CMD` passam a operar nesse contexto, salvo indicação contrária.

---

## `COPY`

Copia arquivos da máquina de build para dentro da imagem.

Exemplo:

```dockerfile
COPY . .
```

Ou de forma mais explícita:

```dockerfile
COPY package.json .
COPY src ./src
```

### Atenção

O `COPY` leva arquivos para dentro da imagem no momento do build.
Ele não é o mesmo que montar volume em tempo de execução.

---

## `RUN`

Executa comandos durante a construção da imagem.

Exemplo:

```dockerfile
RUN npm install
```

Ou:

```dockerfile
RUN apt-get update && apt-get install -y curl
```

### Regra prática

Use `RUN` para preparar a imagem:

* instalar pacotes
* baixar dependências
* ajustar permissões
* criar diretórios necessários

---

## `CMD`

Define o comando padrão executado quando o container inicia.

Exemplo:

```dockerfile
CMD ["npm", "start"]
```

Se o usuário iniciar o container sem informar outro comando, esse será o padrão usado.

---

## `ENTRYPOINT`

Define o executável principal do container.

Exemplo:

```dockerfile
ENTRYPOINT ["python"]
```

Combinado com:

```dockerfile
CMD ["app.py"]
```

Nesse caso, o container tende a executar algo equivalente a:

```bash
python app.py
```

### Diferença prática entre `CMD` e `ENTRYPOINT`

* `CMD` → comando padrão mais fácil de sobrescrever
* `ENTRYPOINT` → executável principal, normalmente mais fixo

---

## `ENV`

Define variáveis de ambiente dentro da imagem.

Exemplo:

```dockerfile
ENV APP_ENV=production
```

Isso pode ser útil para:

* comportamento da aplicação
* configuração de runtime
* flags simples de ambiente

---

## `EXPOSE`

Documenta a porta que a aplicação usa dentro do container.

Exemplo:

```dockerfile
EXPOSE 8080
```

### Atenção

`EXPOSE` não publica a porta sozinho.
Para publicar de fato, ainda é preciso usar `-p` no `docker run` ou configurar isso no Compose.

---

## 5) Como construir uma imagem

Depois de criar o `Dockerfile`, use:

```bash
docker build -t minha-app .
```

### O que esse comando faz

* `build` → inicia o processo de construção
* `-t minha-app` → atribui nome/tag à imagem
* `.` → define o contexto de build como o diretório atual

### Exemplo com tag

```bash
docker build -t minha-app:1.0 .
```

### Regra prática

Use tags explícitas quando quiser mais previsibilidade:

* `minha-app:1.0`
* `minha-app:dev`
* `minha-app:prod`

---

## 6) Como executar a imagem construída

Depois do build:

```bash
docker run -d -p 8000:8000 --name app-local minha-app
```

Isso cria um container a partir da sua imagem customizada.

### Fluxo mental

* `Dockerfile` define
* `docker build` constrói
* `docker run` executa

---

## 7) O que é o contexto de build?

O **contexto de build** é o conjunto de arquivos que o Docker pode enxergar durante o `build`.

No comando:

```bash
docker build -t minha-app .
```

o ponto `.` indica que o contexto é o diretório atual.

### Por que isso importa?

Porque tudo que for copiado com `COPY` ou `ADD` precisa estar dentro desse contexto.

### Erro comum

Tentar copiar arquivos que estão fora da pasta usada como contexto.

---

## 8) Como evitar copiar arquivos desnecessários

Para isso, use um arquivo `.dockerignore`.

Exemplo:

```text
node_modules
.git
.env
__pycache__
*.log
dist
build
```

### Por que isso é importante?

Porque ajuda a:

* reduzir o tamanho do contexto
* acelerar build
* evitar copiar arquivos desnecessários
* proteger conteúdo sensível ou irrelevante

### Regra prática

Se você usa `.gitignore`, provavelmente também deve pensar em um `.dockerignore`.

---

## 9) Exemplo prático com Node.js

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

### Lógica desse exemplo

Primeiro copia os arquivos de dependência, depois instala, e só então copia o restante do projeto.

Isso ajuda no reaproveitamento de camadas quando o código muda, mas as dependências não.

---

## 10) Exemplo prático com Python

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "app.py"]
```

Esse é um padrão simples e muito usado para APIs e scripts Python.

---

## 11) Entendendo camadas da imagem

Cada instrução do Dockerfile tende a gerar uma **camada**.

Exemplo:

* `FROM`
* `COPY`
* `RUN`
* `COPY`
* `CMD`

Isso importa porque o Docker tenta reaproveitar camadas já construídas quando possível.

### Benefício

Se você muda só o código da aplicação, não precisa necessariamente reinstalar todas as dependências, dependendo da ordem das instruções.

### Regra prática

Organize o Dockerfile para aproveitar cache de build.

---

## 12) Ordem das instruções faz diferença

Veja este exemplo melhor estruturado:

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm install

COPY . .

CMD ["npm", "start"]
```

### Por que essa ordem é boa?

Porque o `npm install` só precisa rodar de novo se os arquivos de dependência mudarem.

Se você copiasse tudo antes:

```dockerfile
COPY . .
RUN npm install
```

qualquer alteração simples no código já invalidaria essa camada com mais frequência.

---

## 13) Boas práticas para imagens menores e mais limpas

### Preferir imagens base mais enxutas quando fizer sentido

Exemplo:

* `python:3.12-slim`
* `node:20-alpine`

### Evitar instalar ferramentas desnecessárias

Quanto mais coisa na imagem:

* maior o tamanho
* maior a superfície de ataque
* mais lenta a transferência

### Usar `.dockerignore`

Evita enviar arquivos inúteis para o build.

### Combinar comandos de instalação quando fizer sentido

Exemplo:

```dockerfile
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
```

Isso ajuda a manter a imagem mais limpa.

### Não guardar segredos no Dockerfile

Evite embutir:

* senhas
* tokens
* chaves
* `.env` sensível

---

## 14) Evite confundir build com execução

Esse é um ponto muito importante.

### Durante o build

Você monta a imagem:

* copia arquivos
* instala dependências
* prepara o ambiente

### Durante a execução

Você roda o container:

* publica portas
* monta volumes
* injeta variáveis de ambiente
* conecta redes

### Regra prática

* `Dockerfile` prepara a imagem
* `docker run` define o comportamento da execução

---

## 15) Como sobrescrever o comando padrão

Mesmo que a imagem tenha `CMD`, você pode passar outro comando no `docker run`.

Exemplo:

```bash
docker run minha-app python outro_script.py
```

Isso pode substituir o comando padrão, dependendo de como a imagem foi definida.

Esse comportamento é especialmente comum quando a imagem usa `CMD`.

---

## 16) Testando e depurando sua imagem

Depois do build, alguns comandos úteis:

### Ver imagem criada

```bash
docker images
```

### Rodar container

```bash
docker run --rm minha-app
```

### Ver logs

```bash
docker logs nome-do-container
```

### Abrir shell no container

```bash
docker exec -it nome-do-container sh
```

Esses comandos ajudam a validar:

* se os arquivos foram copiados corretamente
* se as dependências foram instaladas
* se o comando inicial funciona
* se a aplicação realmente sobe

---

## 17) Erros comuns de quem está começando

### Copiar tudo sem controle

Usar `COPY . .` sem `.dockerignore` pode inflar muito a imagem.

### Instalar dependências em ordem ruim

Isso quebra reaproveitamento de cache e deixa builds mais lentos.

### Confundir `EXPOSE` com publicação de porta

`EXPOSE` documenta; quem publica é o `-p`.

### Colocar segredo dentro do Dockerfile

Isso é uma prática ruim de segurança.

### Usar imagem base genérica demais sem necessidade

Pode gerar imagens maiores do que o necessário.

### Não testar a imagem construída

Build bem-sucedido não garante que a aplicação funciona corretamente.

---

## 18) Exemplo de fluxo completo

### Criar o Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "app.py"]
```

### Construir a imagem

```bash
docker build -t app-python:1.0 .
```

### Executar a imagem

```bash
docker run -d -p 8000:8000 --name app-python-local app-python:1.0
```

### Ver logs

```bash
docker logs app-python-local
```

Esse já é um fluxo real de empacotamento de aplicação.

---

## 19) Regra mental para memorizar

Pense assim:

* `FROM` → de onde a imagem começa
* `WORKDIR` → onde os comandos operam
* `COPY` → o que entra na imagem
* `RUN` → o que é executado no build
* `ENV` → variáveis da imagem
* `EXPOSE` → porta documentada
* `CMD` → comando padrão
* `ENTRYPOINT` → executável principal

E no uso:

* `docker build` → constrói
* `docker run` → executa

---

## 20) Quando este tópico está bem dominado?

Você já entendeu bem este tópico quando consegue:

* ler um Dockerfile e entender o que ele faz
* criar uma imagem simples da sua aplicação
* organizar a ordem das instruções
* reduzir desperdício com `.dockerignore`
* executar a imagem gerada com segurança
* diferenciar build de runtime
