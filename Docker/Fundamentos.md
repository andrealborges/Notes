# Docker — Fundamentos (o que é e por que usar)

Este tópico apresenta a base do **Docker** de forma prática. O objetivo é responder:  
> “O que exatamente o Docker faz e por que ele é tão usado no dia a dia de desenvolvimento, dados e infraestrutura?”

Antes de entrar em comandos, Dockerfile e Compose, você precisa entender os conceitos que sustentam todo o uso da ferramenta.

---

## 1) O que é Docker?

O **Docker** é uma plataforma que permite **empacotar, distribuir e executar aplicações em ambientes isolados**, chamados de **containers**.

Na prática, ele resolve um problema muito comum:
> “Na minha máquina funciona, mas no servidor não.”

Com Docker, você define o ambiente da aplicação de forma padronizada, incluindo:
- sistema base
- dependências
- bibliotecas
- variáveis de ambiente
- comandos de inicialização

Assim, a aplicação roda com muito mais consistência em diferentes máquinas.

---

## 2) O que é um container?

Um **container** é uma instância em execução de uma imagem Docker.

Ele funciona como um ambiente isolado para rodar um processo ou aplicação, sem precisar de uma máquina virtual completa.

### Características principais
- é **leve**
- inicializa rápido
- compartilha o kernel do sistema operacional hospedeiro
- isola processos, rede e sistema de arquivos
- pode ser criado e removido com facilidade

### Exemplo prático
Você pode rodar:
- um banco PostgreSQL em um container
- uma API Python em outro
- um frontend Node em outro

Tudo isso na mesma máquina, de forma isolada e organizada.

---

## 3) O que é uma imagem?

Uma **imagem** é o “molde” usado para criar containers.

Ela contém tudo o que a aplicação precisa para rodar:
- sistema base
- pacotes instalados
- dependências
- arquivos da aplicação
- instruções de inicialização

### Regra prática
- **Imagem** = modelo pronto
- **Container** = imagem em execução

### Exemplo
A imagem `nginx` pode ser usada para criar um container com um servidor web Nginx rodando.

---

## 4) Docker não é máquina virtual

Essa é uma das distinções mais importantes.

### Máquina virtual
- virtualiza um sistema operacional completo
- tem kernel próprio
- costuma ser mais pesada
- consome mais memória e processamento
- demora mais para iniciar

### Docker / container
- compartilha o kernel do hospedeiro
- isola apenas o necessário
- é mais leve
- sobe mais rápido
- é ideal para aplicações e serviços

### Regra prática
- **VM** = isolamento mais pesado, infraestrutura completa
- **Container** = isolamento leve, foco em aplicação

---

## 5) Principais conceitos do Docker

Para usar Docker bem, você precisa dominar estes conceitos:

### Imagem
Pacote imutável com tudo necessário para rodar a aplicação.

### Container
Instância em execução de uma imagem.

### Dockerfile
Arquivo de instruções usado para construir uma imagem personalizada.

### Volume
Mecanismo para persistir dados fora do ciclo de vida do container.

Exemplo de uso:
- banco de dados
- uploads
- arquivos de configuração
- compartilhamento com a máquina local

### Rede
Permite a comunicação entre containers e entre container e host.

Exemplo:
- uma API se conectando a um banco em outro container

### Registry
Repositório de imagens Docker.

Exemplos:
- Docker Hub
- GitHub Container Registry
- Azure Container Registry

---

## 6) Por que usar Docker?

Docker é útil porque melhora a **padronização**, a **portabilidade** e a **reprodutibilidade** do ambiente.

### Benefícios principais
- evita diferença entre ambiente local, homologação e produção
- facilita onboarding de novos desenvolvedores
- simplifica instalação de dependências
- acelera testes e validações
- facilita subir serviços auxiliares como banco, cache e mensageria
- melhora a consistência em pipelines e deploys

### Exemplos práticos
Docker é muito útil para:
- subir banco local sem instalar no sistema
- rodar aplicações com dependências específicas
- testar serviços isoladamente
- criar ambientes reproduzíveis para equipes
- padronizar desenvolvimento e entrega

---

## 7) Quando Docker faz mais sentido?

Docker vale muito a pena quando você precisa de:

- ambiente padronizado para times
- rodar múltiplos serviços localmente
- isolar dependências
- simular produção localmente
- evitar instalar muitas ferramentas direto no sistema operacional
- distribuir aplicações com mais previsibilidade

### Casos comuns
- API + banco + cache
- projetos em Python, Node, Java, Go
- pipelines CI/CD
- aplicações com serviços dependentes
- laboratórios de testes e automação

---

## 8) O que Docker não resolve sozinho?

Apesar de ser muito útil, Docker não resolve tudo sozinho.

Ele **não substitui**:
- boas práticas de arquitetura
- observabilidade
- segurança bem configurada
- orquestração em escala
- gestão de infraestrutura

Além disso, Docker não elimina a necessidade de entender:
- portas
- volumes
- redes
- variáveis de ambiente
- permissões de arquivos

---

## 9) Erros comuns de quem está começando

### Achar que container é máquina virtual
Container não é um sistema operacional completo.

### Guardar dados importantes dentro do container
Se o container for removido, os dados podem ser perdidos sem volume.

### Confundir imagem com container
Imagem é o modelo; container é a execução.

### Expor portas sem entender o mapeamento
Uma aplicação dentro do container pode estar em uma porta diferente da porta publicada na máquina.

### Instalar tudo no container sem organização
Imagens mal construídas ficam grandes, lentas e difíceis de manter.

---

## 10) Regra mental para começar bem

Pense em Docker assim:

- **Dockerfile** define como montar o ambiente
- **imagem** é o pacote pronto
- **container** é a aplicação rodando
- **volume** guarda dados persistentes
- **rede** conecta serviços
- **registry** armazena imagens

Se essa lógica estiver clara, os próximos tópicos ficam muito mais fáceis.

---

## Resumo do tópico

Neste ponto, o essencial é entender que Docker serve para **empacotar e executar aplicações em ambientes isolados e reproduzíveis**.

A base conceitual é:

- **imagem** = modelo
- **container** = execução
- **Dockerfile** = receita
- **volume** = persistência
- **rede** = comunicação
- **registry** = distribuição

Com isso claro, o próximo passo natural é aprender a **instalar, configurar e validar o ambiente Docker corretamente**.