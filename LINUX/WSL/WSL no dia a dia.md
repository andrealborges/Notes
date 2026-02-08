# WSL — Uso no Dia a Dia (produtividade real)

Este tópico sai da teoria e entra no **uso prático** do WSL. O objetivo é responder:  
> “Quando eu realmente ganho produtividade usando WSL no lugar das ferramentas nativas do Windows?”

Aqui estão decisões, comandos e configurações que fazem diferença no dia a dia de quem desenvolve, trabalha com dados ou automação.

---

## 1) Quando vale a pena usar WSL em vez de PowerShell?

Use **WSL** quando você precisa de:

- Ferramentas **nativas de Linux** (bash, grep, awk, sed, make, rsync, ssh).
- Scripts `.sh` reais (sem adaptação).
- Ambiente semelhante a **produção** (Linux).
- Docker, pipelines, Python, Node, Spark, etc.
- Comportamento previsível de permissões, paths e shells.

Use **PowerShell** quando você precisa de:

- Automação específica do Windows (registro, serviços, Active Directory).
- Integração profunda com APIs do Windows.
- Scripts `.ps1` focados em administração do SO.

### Regra prática
- **Dev / Dados / Containers / Scripts** → WSL  
- **Administração do Windows** → PowerShell  

---

## 2) Comandos Linux mais úteis no dia a dia

Esses comandos viram “extensão do seu cérebro” no WSL.

### Navegação e arquivos
```bash
ls -la
cd
pwd
tree
```

### Busca e filtragem

```bash
grep "erro" arquivo.log
grep -R "TODO" .
find . -name "*.py"
```

### Manipulação de texto

```bash
cat arquivo.txt
less arquivo.txt
head -n 20 arquivo.txt
tail -f logs.txt
awk '{print $1}' arquivo.csv
sed 's/old/new/g' arquivo.txt
```

### Processos e sistema

```bash
ps aux
top
htop
df -h
du -sh *
free -h
```

### Rede

```bash
curl https://example.com
wget https://arquivo.zip
ssh usuario@servidor
```

---

## 3) Posso rodar scripts `.sh` direto do Windows?

Sim. Existem **duas formas principais**.

### 1️⃣ Rodar pelo próprio WSL (mais comum)

No terminal WSL:

```bash
chmod +x script.sh
./script.sh
```

Ou:

```bash
bash script.sh
```

### 2️⃣ Rodar a partir do PowerShell

Você pode chamar o WSL diretamente:

```powershell
wsl bash script.sh
```

Ou com caminho absoluto:

```powershell
wsl bash /home/andre/scripts/script.sh
```

Isso é ótimo para:

* pipelines locais
* automações híbridas
* integração com scripts Windows

---

## 4) Posso chamar comandos Linux a partir do PowerShell?

Sim — e isso é **muito poderoso**.

### Exemplos simples

```powershell
wsl ls -la
wsl uname -a
wsl date
```

### Executar comandos mais complexos

```powershell
wsl bash -lc "cd /home/andre && python3 script.py"
```

### Capturar saída do Linux no PowerShell

```powershell
$files = wsl ls /home/andre
echo $files
```

Isso permite:

* misturar automações Windows + Linux
* usar Linux como “engine” de processamento

---

## 5) Como abrir arquivos do WSL no VS Code?

### Opção 1️⃣ Abrir a pasta atual

No terminal WSL:

```bash
code .
```

Isso abre o VS Code **conectado ao WSL**, não ao Windows.

> Importante: o VS Code usa a extensão **Remote – WSL** automaticamente.

### Opção 2️⃣ Abrir pelo Explorer

No WSL:

```bash
explorer.exe .
```

Ou no Windows:

* Explorer → `\\wsl$\Ubuntu\home\andre\`

Depois, abra a pasta no VS Code.

---

## 6) Como configurar o terminal (Windows Terminal + WSL)?

O **Windows Terminal** é o melhor terminal para usar WSL.

### Configuração básica

* Instale o **Windows Terminal**
* Ele detecta automaticamente as distros WSL
* Você pode abrir abas com:

  * PowerShell
  * Command Prompt
  * Ubuntu (WSL)

### Tornar WSL o perfil padrão

No Windows Terminal:

* Settings → Startup
* Default profile → selecione sua distro (ex.: Ubuntu)

### Customização útil

* Fonte: Cascadia Code / Fira Code
* Atalhos:

  * `Ctrl + Shift + D` → nova aba
  * `Alt + Shift + +` → split

---

## 7) Como definir o WSL como shell padrão no VS Code?

### Passo 1️⃣ Instale a extensão

* **Remote – WSL** (oficial da Microsoft)

### Passo 2️⃣ Abrir projeto no WSL

Sempre use:

```bash
code .
```

Ou no VS Code:

* `Ctrl + Shift + P`
* “Remote-WSL: Open Folder in WSL”

### Conferir se está correto

No VS Code, no canto inferior esquerdo, deve aparecer algo como:

```
WSL: Ubuntu
```

### Terminal integrado

O terminal interno do VS Code passará a ser:

* bash/zsh do WSL
* Python do WSL
* Node do WSL

---

## 8) Como funciona o `wsl.conf`?

`wsl.conf` é um **arquivo de configuração da distro Linux**, não do Windows inteiro.

Local:

```bash
/etc/wsl.conf
```

Ele controla:

* montagem de discos Windows
* comportamento de interop
* automount
* paths

### Exemplo básico

```ini
[automount]
enabled = true
root = /mnt/
options = "metadata,umask=22,fmask=11"

[interop]
enabled = true
appendWindowsPath = true
```

Após alterar:

```powershell
wsl --shutdown
```

E reabra o WSL.

---

## 9) Como configurar timezone, locale e idioma?

### Timezone

Ver timezone atual:

```bash
timedatectl
```

Definir timezone:

```bash
sudo timedatectl set-timezone America/Sao_Paulo
```

### Locale (idioma e formatação)

Ver locales disponíveis:

```bash
locale -a
```

Gerar locale pt_BR:

```bash
sudo locale-gen pt_BR.UTF-8
```

Definir como padrão:

```bash
sudo update-locale LANG=pt_BR.UTF-8
```

Verificar:

```bash
locale
```

### Idioma do sistema

Normalmente você mantém:

* sistema em inglês (logs, mensagens)
* timezone e formatos locais (pt_BR)

Isso evita problemas em logs, parsing e libs.

---

## Erros comuns no dia a dia (e como evitar)

* ❌ Trabalhar em projetos grandes dentro de `/mnt/c`

  * ✅ Use `/home/usuario`
* ❌ Abrir VS Code no Windows apontando para pasta Linux

  * ✅ Use `code .` direto do WSL
* ❌ Misturar Python do Windows com Python do WSL

  * ✅ Use sempre o Python do ambiente onde o projeto roda
* ❌ Esquecer de reiniciar WSL após mudar config

  * ✅ `wsl --shutdown`