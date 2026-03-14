# Nmap 

nmap -Pn -A -v

# metasploit 

multi/handler #fazer shell dentro do msfconsole#
multi/manage/shell_to_meterpreter #melhorar a shell#
post/multi/recon/local_exploit_suggester #sugestão para exploit local# 


# Python 
## melhorar shell de Script

Python 3: python3 -c 'import pty; pty.spawn("/bin/bash")'
Python 2: python -c 'import pty; pty.spawn("/bin/sh")'
Alternative: python -c 'import os; os.system("/bin/bash")'
