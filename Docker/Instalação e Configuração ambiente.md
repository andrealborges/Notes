# Docker — Instalação e configuração do ambiente

Este tópico sai da teoria e entra na preparação do ambiente. O objetivo é responder:  
> “O que eu preciso instalar e configurar para começar a usar Docker de forma estável no dia a dia?”

Aqui estão os passos, validações e cuidados mais importantes para deixar o Docker pronto para uso em desenvolvimento, dados, automação e containers locais.

---

## 1) O que precisa estar instalado?

Para usar Docker, você precisa de um ambiente capaz de:
- baixar imagens
- criar e executar containers
- expor portas
- persistir dados com volumes
- integrar o terminal com o engine do Docker

A forma mais comum depende do sistema operacional:

### No Windows
O mais comum é usar **Docker Desktop**.

---

## 2) Windows: como normalmente se instala

No Windows, a instalação mais prática costuma ser com **Docker Desktop**, especialmente para quem já usa:
- WSL
- VS Code
- Git
- terminais Linux
- desenvolvimento local com múltiplos serviços

### Pré-requisitos mais comuns
- Windows 10 ou 11
- virtualização habilitada na BIOS/UEFI
- WSL 2 instalado
- recurso de virtualização funcionando corretamente

### Regra prática
Se você estiver no Windows, o cenário mais produtivo costuma ser:

- **Docker Desktop + WSL 2**

Isso tende a oferecer melhor compatibilidade com ambientes modernos de desenvolvimento.

---

## 3) Linux: como normalmente se instala

No Linux, o caminho mais comum é instalar:
- Docker Engine
- Docker CLI
- Docker Compose plugin

Esse cenário costuma ser mais direto porque o Docker roda nativamente no sistema.

### Vantagens comuns no Linux
- menos camadas de abstração
- integração natural com shell e filesystem
- bom desempenho
- ambiente muito próximo de servidores Linux

---

## 4) Como validar se a instalação funcionou

Depois de instalar, a primeira etapa é validar se o Docker está respondendo.

### Comandos principais

```bash
docker --version
```

Esse comando verifica se o cliente Docker está instalado.

```bash
docker version
```

Esse mostra informações do cliente e do servidor Docker.

```bash
docker info
```

Esse mostra detalhes do ambiente, incluindo:

* engine
* storage driver
* containers
* imagens
* recursos disponíveis

---

## 5) Primeiro teste real

O teste clássico é executar um container simples para verificar se o engine está funcionando:

```bash
docker run hello-world
```

Se tudo estiver correto, o Docker:

* baixa a imagem `hello-world` (caso ainda não exista localmente)
* cria um container
* executa o processo
* mostra uma mensagem confirmando que o ambiente está funcionando

### O que esse teste valida

* comunicação com o daemon do Docker
* download de imagens
* criação de containers
* execução básica

---

## 6) Validando se há containers e imagens

Depois dos primeiros testes, vale conferir o estado do ambiente.

### Ver containers em execução

```bash
docker ps
```

### Ver todos os containers, inclusive os parados

```bash
docker ps -a
```

### Ver imagens baixadas

```bash
docker images
```

Esses comandos ajudam a confirmar que o Docker está realmente criando e armazenando os recursos esperados.

---

## 7) Configuração importante no Windows com WSL 2

Se você usa Windows, um dos pontos mais importantes é entender a integração com o WSL 2.

### Por que isso importa?

Porque boa parte da produtividade no Windows vem de usar:

* terminal Linux
* paths Linux
* ferramentas nativas de shell
* arquivos do projeto dentro do filesystem do WSL

### Regra prática

No Windows, o cenário mais estável para desenvolvimento costuma ser:

* abrir o projeto dentro do **WSL**
* rodar comandos Docker a partir desse ambiente
* evitar trabalhar com projetos pesados diretamente em caminhos como `C:\` quando o fluxo depende muito de Linux

---

## 8) Onde guardar os projetos no Windows

Esse ponto faz muita diferença em desempenho.

### Melhor cenário para projetos com Docker + WSL

Guardar os arquivos dentro do filesystem Linux do WSL, por exemplo em algo como:

```bash
/home/seu-usuario/projeto
```

### Cenário menos eficiente

Trabalhar em diretórios montados do Windows, como:

```bash
/mnt/c/Users/...
```

### Por que isso importa?

Quando você trabalha em paths montados do Windows dentro do WSL, pode ter:

* lentidão de leitura/escrita
* pior desempenho com muitos arquivos
* problemas em ferramentas de build
* comportamento menos fluido com Node, Python e containers

### Regra prática

* **Projeto Linux / Docker / Node / Python / dados** → prefira filesystem do WSL
* **Arquivos estritamente Windows** → use filesystem do Windows

---

## 9) Permissões e acesso ao Docker

Outro ponto importante é garantir que seu usuário consiga executar comandos Docker sem atrito.

### No Linux

Em muitos casos, o Docker exige permissões adequadas para acessar o daemon.

Um ajuste comum é adicionar o usuário ao grupo `docker`.

Exemplo:

```bash
sudo usermod -aG docker $USER
```

Depois disso, normalmente é necessário encerrar e abrir a sessão novamente.

### Atenção

Executar tudo com `sudo` pode funcionar, mas não costuma ser o fluxo mais confortável para o dia a dia.

---

## 10) Configuração do Docker Compose

Hoje, o Compose normalmente é usado como plugin do Docker.

Para validar:

```bash
docker compose version
```

Note que o padrão moderno é:

```bash
docker compose
```

e não mais necessariamente:

```bash
docker-compose
```

### Exemplo

```bash
docker compose up
```

Esse detalhe é importante para evitar confusão entre documentações antigas e ambientes atuais.

---

## 11) O que verificar antes de começar a usar de verdade

Antes de seguir para Dockerfile e Compose, vale confirmar se tudo está funcionando neste checklist:

* Docker instalado
* engine respondendo
* `docker run hello-world` funcionando
* imagens sendo baixadas
* containers sendo criados
* `docker compose version` funcionando
* terminal escolhido definido
* local dos projetos bem escolhido
* integração com WSL funcionando, se estiver no Windows

---

## 12) Erros comuns de instalação e configuração

### Docker instalado, mas daemon não responde

Exemplo de sintoma:

```bash
Cannot connect to the Docker daemon
```

Isso normalmente indica que o serviço do Docker não está ativo ou acessível.

---

### Virtualização desabilitada no Windows

Sem virtualização habilitada, Docker Desktop e WSL 2 podem não funcionar corretamente.

---

### WSL 2 não configurado corretamente

No Windows, muitos problemas vêm de uma base WSL incompleta ou mal configurada.

---

### Compose não reconhecido

Às vezes o Docker funciona, mas o Compose plugin não está disponível ou não foi instalado corretamente.

---

### Trabalhar em diretórios errados

Guardar projetos em locais pouco adequados pode gerar lentidão e comportamento inconsistente.

---

## 13) Decisões práticas que melhoram o dia a dia

### Escolha um terminal principal

Defina um terminal padrão para o fluxo de trabalho, por exemplo:

* WSL
* PowerShell
* terminal do VS Code

### Mantenha um fluxo consistente

Evite alternar sem necessidade entre:

* paths Windows
* paths Linux
* shells diferentes

### Teste o ambiente cedo

Antes de começar um projeto real, valide:

* download de imagens
* execução de container
* mapeamento de porta
* persistência com volume

---

## 14) Exemplo mínimo de validação prática

Aqui vai uma sequência simples para confirmar que o ambiente está saudável:

```bash
docker --version
docker version
docker info
docker run hello-world
docker images
docker ps -a
docker compose version
```

Se essa sequência funcionar, o ambiente já está pronto para avançar.

---

## 15) Regra mental para este tópico

Pense na instalação e configuração assim:

* instalar o Docker é só o começo
* o importante é validar que o engine funciona
* no Windows, WSL 2 faz muita diferença
* o local dos arquivos impacta desempenho
* permissões e Compose precisam estar corretos
* um ambiente bem configurado evita grande parte dos problemas futuros

---

## Resumo do tópico

Neste ponto, o essencial é deixar o ambiente Docker funcional, validado e confortável para uso no dia a dia.

A base prática é:

* instalar a ferramenta correta para seu sistema
* validar cliente e servidor Docker
* testar com `hello-world`
* confirmar suporte a `docker compose`
* configurar bem o terminal e os caminhos dos projetos
* no Windows, aproveitar corretamente o WSL 2

Com isso pronto, o próximo passo natural é aprender os **comandos essenciais do dia a dia**, que é onde o uso do Docker realmente começa.
