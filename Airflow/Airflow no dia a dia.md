# Airflow — Comandos e fluxo essenciais do dia a dia

Este tópico sai da instalação e entra no uso real do Airflow. O objetivo é responder:
> "Quais comandos e telas eu realmente preciso saber para operar DAGs no dia a dia?"

Aqui estão os comandos de CLI mais importantes, a navegação essencial pela UI e os fluxos mais comuns para disparar, acompanhar e depurar execuções.

---

## 1) Como pensar no dia a dia do Airflow

Na prática, o ciclo mais comum é:

1. escrever/atualizar uma DAG
2. verificar se ela foi carregada sem erro
3. disparar (manual ou automaticamente pelo `schedule`)
4. acompanhar execução na UI ou via CLI
5. investigar logs em caso de falha
6. reprocessar (backfill) se necessário

Se essa sequência fizer sentido, os comandos ficam muito mais fáceis de memorizar.

---

## 2) Listando DAGs disponíveis

```bash
airflow dags list
```

Esse comando mostra todas as DAGs que o Airflow conseguiu carregar, junto com o arquivo de origem.

### Regra prática
Se uma DAG não aparece aqui, normalmente há um erro de importação no arquivo Python.

---

## 3) Verificando erros de importação

```bash
airflow dags list-import-errors
```

Esse comando é um dos mais úteis para depuração: mostra exatamente qual arquivo de DAG está com erro e qual é o erro.

---

## 4) Pausando e despausando uma DAG

Por padrão, DAGs novas geralmente sobem pausadas.

### Despausar

```bash
airflow dags unpause meu_dag_id
```

### Pausar

```bash
airflow dags pause meu_dag_id
```

### Regra prática
Uma DAG pausada não é disparada automaticamente pelo Scheduler, mesmo que o `schedule` esteja configurado.

---

## 5) Disparando uma DAG manualmente

```bash
airflow dags trigger meu_dag_id
```

Isso cria uma execução (`DAG Run`) imediatamente, fora do agendamento normal.

### Quando usar
- testar uma DAG sem esperar o horário programado
- reprocessar algo pontualmente
- validar uma alteração recente

---

## 6) Testando uma task isoladamente

```bash
airflow tasks test meu_dag_id minha_task 2026-08-01
```

Esse comando executa **uma task específica**, para uma data específica, sem registrar o resultado no banco de metadados.

### Regra prática
Use `tasks test` para depurar rapidamente a lógica de uma task, sem afetar o histórico real de execuções.

---

## 7) Listando tasks de uma DAG

```bash
airflow tasks list meu_dag_id
```

Mostra todas as tasks definidas dentro de uma DAG específica.

---

## 8) Verificando o estado do Scheduler

```bash
airflow jobs check --job-type SchedulerJob
```

Útil para confirmar que o Scheduler está realmente ativo e processando DAGs.

---

## 9) Navegando na UI: visão Grid

A tela **Grid** (antiga "Tree View") mostra o histórico de execuções de uma DAG ao longo do tempo, com cada task representada por um quadrado colorido conforme o status.

### Cores mais comuns
- verde → sucesso
- vermelho → falha
- amarelo → em retry
- azul claro → em execução
- cinza → não executado / skipped

---

## 10) Navegando na UI: visão Graph

A tela **Graph** mostra a DAG como um grafo visual, com as tasks conectadas pelas dependências definidas no código.

### Quando usar
- entender a ordem de execução
- visualizar dependências complexas
- localizar rapidamente qual task travou o fluxo

---

## 11) Navegando na UI: logs

Clicando em uma task específica (em qualquer execução) e depois em "Logs", você vê a saída completa daquela task naquela execução.

### Regra prática
Logs por task e por execução são um dos recursos mais valiosos do Airflow — cada tentativa (`try`) fica registrada separadamente.

---

## 12) Disparando uma DAG manualmente pela UI

Na tela principal de DAGs, o botão de "play" ao lado do nome da DAG permite disparar uma execução manual, com a opção de passar parâmetros (`config`) em JSON.

---

## 13) Backfill: reprocessando período passado

Backfill é o processo de rodar (ou re-rodar) uma DAG para um intervalo de datas passado.

```bash
airflow dags backfill meu_dag_id \
  --start-date 2026-07-01 \
  --end-date 2026-07-05
```

### Quando usar
- a DAG só foi criada depois de um período que precisava ser processado
- houve uma falha que exige reprocessar dias anteriores
- uma correção de bug exige rodar novamente um intervalo específico

### Atenção
Backfill pode dessa forma disparar múltiplas execuções de uma vez — tenha cuidado com o impacto em sistemas externos (APIs, bancos) que serão chamados novamente.

---

## 14) Limpando o estado de uma task (`clear`)

Se uma task falhou e você corrigiu o problema, pode ser necessário "limpar" o estado dela para que rode novamente.

```bash
airflow tasks clear meu_dag_id -t minha_task -s 2026-08-01 -e 2026-08-01
```

Isso também pode ser feito diretamente pela UI, clicando na task e escolhendo "Clear".

### Regra prática
`clear` não apaga histórico — ele marca a task para ser reexecutada.

---

## 15) Variables e Connections na UI

Na aba **Admin**, dois recursos são muito usados no dia a dia:

### Variables
Pares chave/valor usados para parametrizar DAGs sem hardcode no código.

### Connections
Configurações de acesso a sistemas externos (bancos, APIs, cloud), usadas pelos Hooks e Operators.

### Regra prática
Nunca deixe credenciais hardcoded dentro do código da DAG — use Connections e Variables (idealmente com um backend de secrets em produção).

---

## 16) Exemplo de fluxo completo no dia a dia

### Verificar se a DAG carregou sem erro

```bash
airflow dags list-import-errors
```

### Despausar a DAG

```bash
airflow dags unpause meu_dag_id
```

### Disparar manualmente para testar

```bash
airflow dags trigger meu_dag_id
```

### Acompanhar na UI

```text
http://localhost:8080 → Grid View → clicar na execução
```

### Investigar falha nos logs

```text
clicar na task vermelha → Logs
```

### Reprocessar depois do fix

```bash
airflow tasks clear meu_dag_id -t minha_task -s 2026-08-01 -e 2026-08-01
```

Esse já é um ciclo real de operação no dia a dia.

---

## 17) Comandos que você mais vai usar de verdade

Se tivesse que priorizar os mais importantes, seriam estes:

```bash
airflow dags list
airflow dags list-import-errors
airflow dags trigger
airflow dags pause / unpause
airflow dags backfill
airflow tasks list
airflow tasks test
airflow tasks clear
```

Esses formam o núcleo do uso operacional do Airflow.

---

## 18) Erros comuns de quem está começando

### DAG não aparece na UI
Quase sempre é erro de importação no arquivo Python — confira com `airflow dags list-import-errors`.

### Achar que a DAG vai rodar sozinha assim que criada
Se ela estiver pausada, o Scheduler não dispara execuções automáticas.

### Confundir `tasks test` com execução real
`tasks test` não grava estado no banco de metadados — não substitui uma execução de verdade para fins de histórico.

### Fazer backfill sem pensar no impacto externo
Rodar novamente várias datas pode repetir chamadas a APIs, e-mails, cargas em banco, etc.

### Editar a DAG e esperar mudança instantânea
O Scheduler tem um intervalo de varredura; alterações podem levar alguns segundos para refletir.

---

## 19) Regra mental para memorizar

Pense assim:

- **dags list** → o que existe
- **dags list-import-errors** → o que está quebrado
- **dags trigger** → disparar agora
- **dags pause/unpause** → controla se roda automaticamente
- **dags backfill** → reprocessar período passado
- **tasks test** → depurar uma task isolada
- **tasks clear** → marcar para rodar de novo
- **UI (Grid/Graph/Logs)** → onde você enxerga tudo visualmente

Essa lógica já cobre a maior parte do dia a dia.

---

## Resumo do tópico

Neste ponto, o essencial é dominar o ciclo básico de operação de DAGs:

- verificar se carregaram sem erro
- pausar/despausar
- disparar manualmente
- acompanhar execução na UI
- investigar logs
- reprocessar com `clear` ou `backfill`

Com isso, você já consegue operar Airflow no dia a dia com segurança e entendimento. O próximo passo natural é aprender a **escrever suas próprias DAGs**, que é quando você deixa de apenas operar exemplos prontos e passa a criar seus próprios workflows.
