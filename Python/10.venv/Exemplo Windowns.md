# Exemplo: Criação de Environment com venv (Windows + Python 3.10)

# 1. Verificar versões do Python instaladas

No Windows, utilize o launcher `py`:

```bash
py -0
````

Saída esperada (exemplo):

```text
Installed Pythons:
 - 3.11
 - 3.10
```

---

# 2. Criar o ambiente virtual com Python 3.10

```bash
py -3.10 -m venv venv
```

Isso garante que o ambiente será criado com Python 3.10

---

# 3. Ativar o ambiente

```bash
venv\Scripts\activate
```

Após ativar, o terminal ficará assim:

```text
(venv) C:\meu_projeto>
```

---

# 4. Validar versão do Python no ambiente

```bash
python --version
```

Saída esperada:

```text
Python 3.10.x
```

---

# 5. Instalar bibliotecas

```bash
pip install pandas numpy seaborn
```

---

# 6. Gerar arquivo de dependências

```bash
pip freeze > requirements.txt
```

Exemplo de conteúdo:

```txt
pandas==2.x.x
numpy==1.x.x
seaborn==0.x.x
```

---

# 7. Estrutura do projeto

```text
meu_projeto/
│
├── venv/
├── main.py
├── requirements.txt
└── README.md
```

---

# 8. Testar o ambiente

Crie um arquivo `main.py`:

```python
import pandas as pd
import numpy as np
import seaborn as sns

print("Ambiente configurado com sucesso!")
```

Execute:

```bash
python main.py
```

---

# 9. Recriar ambiente em outra máquina

```bash
py -3.10 -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
```

---

# 10. Desativar ambiente

```bash
deactivate
```