# Nmap 

nmap -Pn -A -v

# metasploit 

- multi/handler #fazer shell dentro do msfconsole#

- multi/manage/shell_to_meterpreter #melhorar a shell#

- post/multi/recon/local_exploit_suggester #sugestão para exploit local# 


# Python 
## melhorar shell de Script

- Python 3: python3 -c 'import pty; pty.spawn("/bin/bash")'

- Python 2: python -c 'import pty; pty.spawn("/bin/sh")'

- Alternative: python -c 'import os; os.system("/bin/bash")'

# privesc
- whoami
- id
- uname -a
- cat /etc/os-release
- cat /etc/crontab
- sudo -l
- find / -perm -4000 -type f 2>/dev/null
- getcap -r / 2>/dev/null
- ps aux | grep root


# WGET

- wget -r -np -nH --cut-dirs=1 # arquivos em modo mirror


---

_No momentomento que a prova iniciar é importante que tenha uma planilha com os seguintes topicos_

``
| Nome do host | IP | Serviços | Credenciais | Vulnerabilidades | Observações |
``

# Pivoteamento 

_Existe mais de um IP na rede DMZ e talvez seja exatemente por isso que seja necessario realizar o pivotiamento._
_Dentro da DMZ tem mais de uma maquina, mas aparentemente so tem uma maquina que tem comunicação com a rede interna, preciso de cobrir como saber que ela qual é e fazer o 'pivo' logo de inicio_
_Para realizar o pivoteamento o melhor cenario é o MSF Routing usando MSF autoroute + portfwd_

---

_Não ignore os outputs do NMAP ou dos outros scanners como SBMmap, crackmapexec ou enum4linux etc... aparentemente as creds podem ser usadas em diversos serviços como por ex.: Houve um relato de que uma pessoa conseguiu as creds do RDP por um scan do nmap que retornou as creds pelo protocolo SMB._
_So com essas informações ja da pra saber que existe um ambiente hibrido de Linux e Windows_

_Prestar atenção em arquivos do tipo .bak .old .save em todas as suas variações, fiz um CTF da propŕia empresa que tinha um arquivo que normalmente é 'wp-config.php' mas como tinha que pegar o bak dele tive que procurar por 'wp-config.bak'_

_Aparentemente as perguntas te guiam para onde esta as flags (talvez o exame seja assim tambem, outro ponto importante é que a flag vem em um formato +/- assim FLAG1{hashMD5} tipo: FLAG1{23c7d4bababf8048c0cda5136ac83c9e} a flag deve ser entregue como: '23c7d4bababf8048c0cda5136ac83c9e' e não dentro das chaves)_

_quando iniciar o teste tenta pegar os hosts que estão configurados no '/etc/hosts' isso pode matar um pouco do tempo de enumeração_


