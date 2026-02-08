# WSL — Arquitetura Interna (como tudo se conecta)

Este tópico explica “o que está rodando onde” quando você usa WSL, como os arquivos são armazenados/compartilhados, como rede e IP funcionam e como Windows e Linux conversam entre si. A ideia é você conseguir diagnosticar problemas com clareza (performance, caminhos, portas, VS Code, Docker, etc.).

> Observação: a maior parte deste tópico assume **WSL 2**, que é o padrão hoje. Quando houver diferença relevante para WSL 1, eu sinalizo.

---

## 1) O que exatamente roda no Windows e o que roda no Linux no WSL?

### No WSL 2 (o mais comum)
Pense assim: **Windows é o “host”**, e o WSL 2 roda um **Linux real** dentro de uma **VM leve** gerenciada pelo Windows.

- **No lado Windows (host):**
  - O Windows (kernel Windows) continua sendo o sistema principal.
  - O gerenciamento do WSL (iniciar/parar distros, integração, montagem de discos, rede, etc.) é feito por componentes do Windows.
  - Você usa apps Windows normalmente (PowerShell, Explorer, VS Code Windows, etc.).

- **No lado Linux (guest do WSL 2):**
  - Uma distro Linux completa (ex.: Ubuntu) com:
    - processos Linux (bash, python, apt, etc.)
    - usuários Linux (ex.: `andre`)
    - permissões Linux (chmod, chown)
    - filesystem Linux (ext4 virtualizado)
  - Um **kernel Linux** real (fornecido/atualizado pelo WSL).

**Resumo mental:**
- Programas Linux → rodam “de verdade” no Linux do WSL 2.
- Programas Windows → rodam no Windows.
- O WSL faz a “ponte” entre os dois mundos.

### Como verificar isso na prática (processos)
No Windows (PowerShell), você pode ver que existe um processo que representa o WSL:

```powershell
wsl --status
wsl -l -v
```

No Linux (WSL), você vê processos Linux normalmente:

```bash
ps aux | head
uname -a
```

`uname -a` no WSL 2 mostra kernel Linux.

---

## 2) Onde fica o filesystem do Linux dentro do Windows?

No WSL 2, o filesystem Linux (ex.: `/home`, `/etc`, `/var`) fica guardado em um **disco virtual** (VHDX) mantido pelo Windows.

Você não precisa (e geralmente não deve) mexer diretamente nesse arquivo VHDX, mas é importante saber que:

* Ele é o “HD” do Linux no WSL.
* Dentro dele existe um filesystem Linux (tipicamente ext4).

### Acesso recomendado ao filesystem Linux

Em vez de caçar VHDX, use:

* Pelo Explorer (Windows): `\\wsl$\`
* Pelo próprio Linux: caminhos normais (`/home/andre`, `/etc`, etc.)

---

## 3) O que é o caminho `\\wsl$` e como ele funciona?

`\\wsl$` é um “compartilhamento de rede” (estilo SMB/UNC path) exposto pelo Windows para você acessar o filesystem Linux **como se fosse uma pasta**.

Exemplo no Explorer:

* `\\wsl$\Ubuntu\home\andre\`

Isso permite:

* abrir arquivos do Linux em apps Windows (VS Code, Notepad, Excel, etc.)
* copiar arquivos facilmente entre ambientes
* navegar na estrutura Linux sem “montar” nada manualmente

### Exemplos práticos

Abrir a home no Explorer, direto pelo WSL:

```bash
explorer.exe .
```

Isso abre a pasta atual do Linux no Explorer (via integração do WSL).

Ou, no Windows, acessar diretamente:

* Explorer → barra de endereço → `\\wsl$\Ubuntu\home\andre\`

---

## 4) Diferença entre `/home/usuario` e `/mnt/c`

Esses dois caminhos são a origem de MUITA confusão e MUITOS problemas de performance.

### `/home/usuario` (filesystem Linux “nativo” do WSL)

* Fica dentro do disco virtual do WSL (ext4).
* Permissões Linux funcionam 100% (chmod/chown).
* Geralmente **melhor performance** para builds, git, pip/conda, node_modules, etc.
* É o lugar mais “correto” para projetos que você roda no WSL.

Exemplo:

```bash
cd /home/andre/projetos/meu_projeto
ls -la
```

### `/mnt/c` (filesystem do Windows montado dentro do Linux)

* É o seu `C:\` do Windows, acessível dentro do WSL.
* Serve para compartilhar arquivos com Windows.
* Pode ser **mais lento** em cenários intensivos (muitos arquivos pequenos).
* Permissões Linux não são “nativas” aqui (são mapeadas).

Exemplo:

```bash
cd /mnt/c/Users/Andre/Documents
ls
```

### Regra prática (bem importante)

* Projetos com muitos arquivos (Python env, node, builds, git pesado) → prefira **/home**.
* Arquivos que você precisa abrir/editar sempre no Windows (Power BI, Office, etc.) → podem ficar no `C:\` (acesso via `/mnt/c`), mas cuidado com performance.

---

## 5) O WSL compartilha CPU, RAM e disco com o Windows?

### CPU

Sim. A VM do WSL 2 usa a **CPU da máquina**, compartilhada com o Windows.

* Se você rodar algo pesado no WSL, o Windows “sente”.

### RAM

Sim, mas com dinâmica diferente:

* O WSL 2 “pega” memória conforme precisa.
* E devolve parte dela conforme o sistema ajusta (hoje melhor do que no início do WSL 2, mas ainda pode variar).

Você pode limitar memória/CPU do WSL via `.wslconfig` no Windows.

Exemplo de arquivo em:
`C:\Users\<seu_usuario>\.wslconfig`

```ini
[wsl2]
memory=8GB
processors=4
swap=4GB
```

Aplicar mudanças:

```powershell
wsl --shutdown
```

### Disco

* O disco do WSL (Linux) fica no VHDX (disco virtual) dentro do Windows.
* O WSL também acessa seu disco Windows via `/mnt/c`, `/mnt/d`, etc.

---

## 6) Como funciona a rede no WSL?

### WSL 2: rede virtualizada (NAT)

No WSL 2, o Linux roda em uma VM e a rede normalmente funciona via **NAT**:

* O Linux tem uma interface de rede virtual.
* O Windows atua como “gateway” para o WSL.

Isso significa:

* O WSL consegue acessar a internet normalmente.
* O Windows consegue acessar serviços do WSL via `localhost` em muitos casos, mas existem nuances (especialmente com firewall, bind de interfaces e casos avançados).

### Ver rede e interfaces no Linux

```bash
ip addr
ip route
```

Testar conectividade:

```bash
ping -c 3 8.8.8.8
ping -c 3 google.com
```

Ver DNS:

```bash
cat /etc/resolv.conf
```

---

## 7) O WSL tem IP próprio?

### WSL 2: normalmente, sim

No WSL 2, como ele roda em VM, ele costuma ter **um IP interno** diferente do Windows.

Ver IP do WSL:

```bash
ip -4 addr show eth0
```

Ver o IP do Windows (do lado Windows):

```powershell
ipconfig
```

### Importante: “localhost” e portas

Em muitos casos:

* serviço no WSL escutando em `0.0.0.0:porta` pode ser acessível do Windows via `localhost:porta`
* mas se estiver escutando só em `127.0.0.1` (loopback do Linux), pode causar confusão dependendo do cenário

Exemplo de servidor no WSL:

```bash
python3 -m http.server 8000 --bind 0.0.0.0
```

E no Windows:

* abrir `http://localhost:8000`

---

## 8) Como funciona a comunicação entre processos Windows ↔ Linux?

Essa é uma das partes mais legais do WSL: **chamar coisas de um lado e do outro**.

### Chamando programas Windows a partir do Linux

No WSL, você consegue rodar executáveis Windows:

```bash
notepad.exe README.md
explorer.exe .
code .
```

Também funciona para caminhos do Windows (com cuidado):

```bash
explorer.exe /mnt/c/Users/Andre
```

### Chamando programas Linux a partir do Windows

No PowerShell, você pode executar comandos Linux diretamente:

```powershell
wsl ls -la
wsl uname -a
wsl bash -lc "python3 --version"
```

### Passando dados entre os mundos

Exemplo: pegar data do Linux e usar no Windows:

```powershell
$linuxDate = wsl date "+%Y-%m-%d"
echo $linuxDate
```

### Interop de paths (conceito prático)

* Windows → `C:\Users\Andre\...`
* WSL → `/mnt/c/Users/Andre/...`

Dentro do WSL, para converter path:

* dá pra usar `wslpath`:

```bash
wslpath -w /home/andre/projetos
wslpath -u "C:\Users\Andre\Documents"
```

---

## Checklist rápido (para você diagnosticar como engenheiro)

1. “Estou no Linux do WSL?”

```bash
uname -a
```

2. “Estou trabalhando no filesystem Linux (rápido) ou no Windows (potencialmente lento)?”

```bash
pwd
# /home/...  (Linux)
# /mnt/c/... (Windows)
```

3. “Qual é meu IP dentro do WSL?”

```bash
ip -4 addr show eth0
```

4. “Um serviço está escutando em todas interfaces?”

```bash
ss -tulpn | head
# procure 0.0.0.0:PORTA
```