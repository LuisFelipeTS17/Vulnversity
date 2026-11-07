# Relatório Técnico de Teste de Intrusão: Lab Vulnversity (TryHackMe)

**Autor:** [@LuisFelipeTS17](https://github.com/LuisFelipeTS17)  
**Alvo:** Máquina Virtual TryHackMe - Vulnversity  
**Nível:** Fácil  

---

## 1. Executive Summary
Este relatório documenta o processo técnico de auditoria e exploração da máquina virtual **Vulnversity** no TryHackMe. O laboratório simulou um ambiente web vulnerável, permitindo demonstrar falhas de arquitetura em servidores web, fraquezas em validação de inputs em formulários de upload e falhas de configuração em permissões especiais de ficheiros (SUID) em sistemas operacionais Linux baseados em Ubuntu. O resultado foi o comprometimento total do alvo, partindo de um acesso não autenticado até privilégios de **root**.

---

## 2. Metodologia e Fases de Execução

### 2.1. Reconhecimento Ativo
O levantamento de portas e serviços foi efetuado em duas etapas: varredura TCP SYN furtiva com detecção de versões (`nmap -sS -sV`) e, em seguida, varredura agressiva (`nmap -A`) para scripts e fingerprint do SO. Foram identificadas **6 portas abertas**, destacando-se a porta `3333`, que hospeda um `Apache httpd 2.4.41` com título `Vuln University`.

### 2.2. Enumeração de Diretórios Web
Utilizou-se a ferramenta **Gobuster** com a wordlist clássica do DirBuster (`directory-list-1.0.txt`) para força bruta de diretórios na porta `3333`:
* **Comando:** `gobuster dir -u http://<IP>:3333 -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt`
* **Descoberta Crítica:** Identificação do diretório `/internal/`, que continha um formulário de carregamento de ficheiros (*file upload form*).

### 2.3. Contorno de Filtros (Bypass) e Execução de Código
A requisição de upload foi interceptada no **Burp Suite** e enviada ao Intruder (ataque *Sniper*) para testar a robustez do filtro de extensões, utilizando a wordlist `phpext.txt`. O servidor bloqueia `.php`, mas aceita **`.phtml`**. Foi carregado um reverse shell PHP (`images.phtml`), e o seu acesso via navegador estabeleceu um shell interativo como `www-data`.

### 2.4. Escalonamento de Privilégios (SUID)
A enumeração de binários SUID (`find / -perm -u=s -type f 2>/dev/null`) revelou `/bin/systemctl`. Através da criação de um serviço `systemd` malicioso que executa `chmod +s /bin/bash`, seguido de `/bin/bash -p`, obteve-se um shell com **EUID 0 (root)**.

---

## 3. Registo Fotográfico (Capturas de Ecrã da Sessão)

As capturas estão organizadas por fase, na pasta `docs/screenshots/`.

### Fase 1 — Reconhecimento (Nmap)

![Nmap — scan de serviços](screenshots/nmap_service_scan.png)
![Nmap — scan furtivo + agressivo](screenshots/nmap_ss_sv_and_a.png)
![Nmap — scan agressivo (-A)](screenshots/nmap_aggressive_scan.png)
![Nmap — visão geral](screenshots/nmap_scan_overview.png)
![Nmap — cheatsheet de flags](screenshots/nmap_flags_cheatsheet.png)
![TryHackMe — respostas do Nmap](screenshots/thm_nmap_answers.png)
![TryHackMe — SO alvo (Ubuntu)](screenshots/thm_os_answer.png)

### Fase 2 — Enumeração de Diretórios (Gobuster)

![Gobuster — /internal descoberto](screenshots/01-gobuster.png)
![TryHackMe — resposta Gobuster correta](screenshots/02-thm-gobuster-correto.png)

### Fase 3 — Comprometimento Web (Upload Bypass → RCE)

![Formulário de upload](screenshots/03-upload-form.png)
![Burp Proxy — interceptação](screenshots/04-burp-intercept.png)
![Burp Intruder — posições (Sniper)](screenshots/05-intruder-positions.png)
![Burp Intruder — payloads](screenshots/06-intruder-payloads.png)
![Burp Intruder — resultados](screenshots/07-intruder-results.png)
![Burp Intruder — resposta](screenshots/08-intruder-response.png)
![Upload bem-sucedido (images.phtml)](screenshots/09-upload-success.png)
![Payload PHP (reverse shell)](screenshots/10-shell-payload.png)
![Netcat — listener](screenshots/11-nc-listening.png)
![Reverse shell como www-data](screenshots/12-reverse-shell.png)

### Fase 4 — Escalonamento de Privilégios (SUID)

![Enumeração SUID](screenshots/14-suid-enum.png)
![Arquivo de serviço systemd](screenshots/15-service-file.png)
![Exploração via systemctl](screenshots/16-systemctl-exploit.png)
![EUID 0 — root obtido](screenshots/17-root-euid.png)
![Flag de root capturada](screenshots/18-root-flag.png)
