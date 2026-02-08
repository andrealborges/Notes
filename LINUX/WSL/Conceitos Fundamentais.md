# WSL — Conceitos Fundamentais (o “o que é” e “como funciona”)

Este tópico explica os conceitos básicos para entender o que é o WSL, por que ele existe e como ele funciona por baixo dos panos. A ideia é que, depois daqui, você consiga responder: “o que exatamente está rodando no meu PC quando eu uso WSL?”.

---

## 1) O que é o WSL e qual problema ele resolve?

**WSL (Windows Subsystem for Linux)** é uma forma de rodar um ambiente Linux dentro do Windows, com integração forte entre os dois sistemas.

Ele resolve principalmente estes problemas:

- **Compatibilidade de ferramentas**: muitas ferramentas de desenvolvimento e dados são “nativas” de Linux (bash, grep, awk, sed, ssh, rsync, make, etc.).
- **Ambiente mais próximo de produção**: a maioria dos servidores e pipelines (cloud, containers, CI/CD) roda Linux. O WSL permite desenvolver no Windows mantendo um ambiente Linux realista.
- **Evitar a dor de manter uma VM completa**: em vez de um “PC inteiro dentro do PC”, o WSL traz um Linux bem integrado, com menos fricção.
- **Evitar dual boot**: você não precisa reiniciar a máquina para alternar entre Windows e Linux.

Na prática, você ganha:
- Terminal Linux (bash/zsh)
- Gerenciador de pacotes (apt, etc.)
- Ferramentas CLI Linux
- Execução de processos Linux lado a lado com Windows

---

## 2) Qual a diferença entre WSL 1 e WSL 2?

### WSL 1 (tradução de chamadas do Linux para Windows)

O WSL 1 funciona como uma “camada de compatibilidade”: ele **traduza chamadas de sistema (syscalls)** Linux para chamadas equivalentes no kernel do Windows.

* ✅ Integração bem direta com arquivos do Windows
* ✅ Leve (sem VM “real”)
* ❌ Nem todas as syscalls existem ou funcionam igual
* ❌ Menos compatível com workloads modernos (especialmente containers e algumas ferramentas)

### WSL 2 (kernel Linux real em uma VM leve)

O WSL 2 roda com um **kernel Linux real**, dentro de uma **VM altamente otimizada** (muito mais integrada e leve do que VMs tradicionais).

* ✅ Compatibilidade muito maior (kernel real)
* ✅ Melhor para Docker/containers
* ✅ Melhor para I/O e workloads Linux “de verdade”
* ⚠️ Acesso ao filesystem do Windows via `/mnt/c` pode ser mais lento do que trabalhar dentro do filesystem Linux
* ⚠️ Usa virtualização (Hyper-V por trás)

### Como verificar sua versão do WSL

No PowerShell:

```powershell
wsl -l -v
```

Saída típica:

```
  NAME            STATE           VERSION
* Ubuntu          Running         2
  Debian          Stopped         2
```

---

## 3) O WSL usa máquina virtual ou não?

Depende da versão:

* **WSL 1**: não usa VM tradicional; é tradução/compatibilidade de syscalls.
* **WSL 2**: usa sim uma **VM leve**, porque precisa rodar um kernel Linux real.

Atenção: quando você lê “VM”, muita gente imagina “pesada e isolada” (VirtualBox). **A VM do WSL 2 é diferente**: ela é gerenciada pelo Windows, integrada, com inicialização rápida e foco em dev.

---

## 4) O que é o kernel Linux e por que o WSL 2 precisa dele?

**Kernel** é o “núcleo” do sistema operacional. Ele é responsável por:

* gerenciar CPU (processos/threads)
* gerenciar memória
* controlar dispositivos (drivers)
* implementar syscalls (as chamadas que programas fazem ao sistema)
* redes, filesystem, permissões, etc.

No Linux, muitas ferramentas e aplicações dependem de syscalls e comportamentos específicos do kernel.
O WSL 2 usa um **kernel Linux real** para garantir que softwares Linux funcionem como funcionariam em um servidor Linux.

Exemplo: containers (Docker) dependem fortemente de recursos do kernel Linux (namespaces, cgroups).
Por isso, o WSL 2 se tornou o padrão recomendado para Docker.

---

## 5) Qual é a relação entre Windows, WSL e Hyper-V?

**Hyper-V** é a tecnologia de virtualização do Windows (um hypervisor).
No WSL 2, o Windows usa a infraestrutura do Hyper-V para rodar a VM leve que contém o kernel Linux.

* WSL 2 → usa virtualização → geralmente via Hyper-V/Virtual Machine Platform
* Docker Desktop (no Windows) → com frequência usa WSL 2 como backend

Você não precisa “criar uma VM” manualmente: o Windows gerencia isso.

### Verificando recursos habilitados (PowerShell)

```powershell
dism.exe /online /get-features /format:table | findstr /i "VirtualMachinePlatform Microsoft-Windows-Subsystem-Linux Hyper-V"
```

---

## 6) O WSL roda Linux “de verdade” ou é uma emulação?

* **WSL 1**: não é emulação de CPU (não é tipo rodar Linux em um emulador), mas é uma camada de compatibilidade (tradução de syscalls). Então ele **não é um Linux completo com kernel real**.
* **WSL 2**: roda Linux “de verdade” no sentido técnico mais importante para dev: **kernel Linux real**, comportamento compatível, syscalls reais. Ele roda dentro de uma VM leve, mas isso não faz dele “fake”.

Uma forma prática de entender:

* Se algo exige recursos do kernel Linux, o **WSL 2 costuma funcionar**; o WSL 1 pode falhar.

---

## 7) Por que o WSL é diferente de uma VM tradicional (VirtualBox/VMware)?

WSL 2 usa VM, mas é diferente por:

1. **Integração com Windows**

* Acesso fácil ao filesystem do Windows (`/mnt/c`)
* Acesso ao filesystem Linux via `\\wsl$\Ubuntu\home\...`
* Chamadas Windows ↔ Linux com facilidade

2. **Inicialização e uso simplificados**

* Você abre um terminal e pronto
* Sem gerenciar ISO, boot, interface de VM, etc.

3. **Objetivo**

* VM tradicional: simular um computador completo
* WSL 2: fornecer um ambiente Linux para dev com integração e produtividade

4. **Consumo dinâmico**

* WSL 2 tende a ajustar uso de memória/recursos de forma mais dinâmica que VMs comuns (na prática, você sente menos “peso fixo”)

---

## 8) O que muda em desempenho entre WSL e dual boot?

### Dual boot

* ✅ Performance “nativa” total do Linux (sem camada extra)
* ✅ Controle completo de drivers, kernel, etc.
* ❌ Precisa reiniciar para trocar de sistema
* ❌ Fluxo de trabalho mais lento para quem usa Windows no dia a dia

### WSL 2

* ✅ Muito rápido e prático (sem reboot)
* ✅ Excelente para dev, Docker, scripts, automação
* ⚠️ Pode haver gargalo em alguns cenários, principalmente:

  * acesso intensivo ao filesystem do Windows via `/mnt/c`
  * workloads que exigem controle profundo do hardware
  * tarefas de kernel/driver específicas

Regra prática:

* **Para desenvolvimento e dados**: WSL 2 geralmente é “o melhor custo-benefício”.
* **Para uso 100% Linux e performance máxima** (ou drivers específicos): dual boot pode ser melhor.