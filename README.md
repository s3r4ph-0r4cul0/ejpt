# 🧠 INE Certification Exam Cheatsheets

---

Este repositório serve como uma central de guias de referência rápida (Cheatsheets), comandos práticos, payloads e táticas para a preparação e execução dos exames de segurança da **INE / eLearnSecurity**.

Os guias são focados na otimização de tempo durante as provas práticas, fornecendo atalhos rápidos e explicações detalhadas para as principais ferramentas e vetores de ataque cobrados.

---

## 📚 Certificações Cobertas

| Exame | Descrição do Foco | Link para o Guia |
|:---:|---|:---:|
| **eJPT** | Testes de invasão em rede, enumeração ativa/passiva, exploração básica de sistemas (Windows/Linux) e pivoteamento simples. | [Acessar Guia do eJPT 🚀](eJPT.md) |
| **eWPT** | Testes de invasão em aplicações web, injeção de SQL, XSS, LFI/RFI, vulnerabilidades de CMS, desvios de filtros e XXE. | [Acessar Guia do eWPT 💉](eWPT) |

---

## 🛠️ Dicas Gerais para os Exames da INE

> [!IMPORTANT]
> A gestão de tempo e a organização dos dados coletados são os fatores mais críticos para aprovação nos exames práticos.

1. **Estruture sua planilha de hosts imediatamente:**
   Assim que a prova iniciar, crie uma planilha com os tópicos abaixo para evitar confusão de alvos:
   
   | IP | Nome do Host | Portas Abertas | Serviços / Versões | Credenciais Encontradas | Flag Obtida? |
   |---|---|---|---|---|:---:|
   | `10.10.x.x` | `alvo1.ine.local` | `80, 22, 445` | `Apache 2.4, OpenSSH 8.2, Samba` | `admin:password` | `[x]` |

2. **Enumere tudo antes de explorar:**
   * Nunca tente um exploit complexo ou força bruta agressivo sem antes realizar uma enumeração completa de portas e banners de serviço.
   * Procure sempre por arquivos de backup expostos (`.bak`, `.old`, `.zip`, `.sql`).

3. **Mapeamento de Hosts locais (`/etc/hosts`):**
   * Se o ambiente de prova utiliza nomes de domínio internos (`target.ine.local`), adicione-os manualmente ao `/etc/hosts` da sua máquina de ataque (Kali Linux).

4. **Upgrade de Shell rápido:**
   Sempre faça o upgrade da sua shell interativa simples para um terminal TTY completo utilizando Python:
   ```bash
   python3 -c 'import pty; pty.spawn("/bin/bash")'
   ```

---

## 🔍 Como Buscar Tópicos no Repositório

Você pode facilmente pesquisar comandos e tópicos específicos em todos os cheatsheets usando o `grep` no terminal:

```bash
# Buscar por um payload ou comando específico (ex: "hydra")
grep -rnw . -e "hydra"

# Buscar arquivos ou referências sobre CMS (ex: "WordPress")
grep -rnw . -e "wpscan"
```

---

## 🤝 Contribuições e Melhorias

Sinta-se à vontade para abrir pull requests ou sugerir novos payloads e dicas práticas à medida que evolui nos seus estudos!

---
*(Desenvolvido para fins educacionais e preparação para exames de certificação oficial).*
