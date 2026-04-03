# Python Environments

# 1. Conceito de Environment

Um **Python Environment** é um ambiente isolado que contém:
- Uma versão específica do Python
- Um conjunto específico de bibliotecas

## Por que usar?
- Evitar conflitos entre dependências
- Permitir múltiplos projetos com versões diferentes
- Garantir reprodutibilidade
- Facilitar deploy

## Ambiente Global vs Virtual

| Tipo | Descrição |
|------|----------|
| Global | Pacotes instalados no sistema inteiro |
| Virtual | Ambiente isolado por projeto |

---

# 2. Tipos de Environments

## 2.1 venv (nativo)
- Simples e leve
- Recomendado para a maioria dos projetos

## 2.2 virtualenv
- Similar ao venv
- Compatível com versões antigas do Python

## 2.3 Conda
- Gerencia Python + dependências nativas (C, libs científicas)
- Ideal para Data Science

---

# 3. Criação e Gerenciamento

## venv

```bash
python -m venv venv
```

Ativar:

```bash
# Linux/Mac
source venv/bin/activate

# Windows
venv\Scripts\activate
```

Desativar:

```bash
deactivate
```

## Conda

Criar ambiente:

```bash
conda create -n meu_env python=3.10
```

Ativar:

```bash
conda activate meu_env
```

Desativar:

```bash
conda deactivate
```

---

# 4. Gerenciamento de Dependências

## Instalar pacotes

```bash
pip install pandas
```

## Arquivos de dependência

### requirements.txt

```txt
pandas==2.0.1
numpy>=1.25
scikit-learn
```

## Exportar dependências

```bash
pip freeze > requirements.txt
```

## Instalar via arquivo

```bash
pip install -r requirements.txt
```

---

# 5. Reprodutibilidade

## Práticas recomendadas

* Fixar versões (`==`)
* Utilizar lock files
* Definir versão do Python

## Exemplo

```txt
pandas==2.0.1
numpy==1.25.0
```

## Problema comum

> "Funciona na minha máquina"

Causa:

* Versões diferentes de bibliotecas

---

# 6. Ambientes em Projetos de Dados e ML

## Separação de ambientes

* Desenvolvimento
* Teste
* Produção

## Integrações

* Jupyter Notebook
* MLflow
* Pipelines de dados

## Riscos

* Drift de dependências
* Diferença entre treino e produção

---

# 7. Ambientes em Cloud / Big Data

## Databricks

* Instalação via `%pip install`
* Bibliotecas por cluster ou notebook

## Microsoft Fabric

* Ambientes gerenciados
* Integração com Lakehouse

## Boas práticas

* Evitar instalar manualmente em produção
* Versionar dependências

---

# 8. Containers vs Environments

| Aspecto    | Environment     | Container        |
| ---------- | --------------- | ---------------- |
| Escopo     | Python          | Sistema completo |
| Isolamento | Médio           | Alto             |
| Uso        | Desenvolvimento | Produção         |

## Quando usar Docker?

* Deploy em produção
* Padronização de ambiente
* Portabilidade



# 9. Estrutura Recomendada de Projeto

```
project/
│
├── src/
├── notebooks/
├── tests/
├── requirements.txt
├── pyproject.toml
└── README.md
```

# 10. Checklist de Criação de Ambiente

* [ ] Criar ambiente virtual
* [ ] Definir versão do Python
* [ ] Instalar dependências
* [ ] Exportar requirements.txt
* [ ] Documentar setup