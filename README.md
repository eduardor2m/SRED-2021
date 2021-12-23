# Instituto Federal de Alagoas - Campus Arapiraca
- Disciplina: SRED
- Turma: 914
- Dupla: Eduardo Roger e Gustavo Eloizio
# Implementar o Servidor Samba e o DNS Na Sua Máquina Virtual.
> Escrever um roteiro de instalação com os dados da sua máquina virtual (plano de nomes e endereçamento IP).
> Ilustrar com prints de tela e sessões de código, conforme o nosso roteiro no github no formato de tutorial.
> Não deve-se somente reproduzir o nosso roteiro no github. Mas criar o seu próprio roteiro de instalação.

# Requisitos
- [x] Cabeçalho com o nome do IFAL, Disciplina, Turma e Nome da dupla
- [x] Introdução sobre o trabalho considerando o que será feito e para que serve os serviços ofertados neste trabalho.
- [x] Nome do domínio, no formato: (nomedupla).labredes.ifalarapiraca.local
- [x] Tabela de nomes FQDN dos hosts
- [x] Tabela de definições de IPs, contendo o gateway, o dns e o samba
- [x] Roteiro de implementação do Samba
- [x] Roteiro de implementação do DNS
- [x] Configuração das interfaces de rede dos hosts da dupla para os nomes que foram planejado na tabela de nomes FQDN
- [x] Sessão de testes contendo todos os testes de dig, nslookup e ping de todos os hosts para todos os hosts usando nomes e IPs
- [x] Sessão de testes gravando um arquivo no samba através de um host local com acesso via VPN ao labredes (Windows, Linux, ChromeOS e etc.)
- [x] Configuração da interface do host local para usar o seu servidor DNS

# Tabelas de Definições de Rede

### Rede Interna

| DESCRIÇÃO   | IP            |
|:------------|:------------- |
| Rede        | 10.9.14.0     |
| Máscara     | 255.255.255.0 |
| VirtualBox (gateway)     | 10.9.14.1      |
| Broadcast   | 10.9.14.255  |
| ns1         | 10.9.14.106   |
| samba       | 10.9.14.106   |

### Rede Externa

|  DESCRIÇÃO  |       IP      |
|:------------|:------------- |
| Rede        | 192.168.0.201 |
| Máscara     | 255.255.255.0 |
| VirtualBox (gateway)     | 192.168.0.1 |
| Broadcast   | 192.168.0.255 |

### Nomes dos Servidores

|    Nome da VM     |                       NOME                           |
|:------------------|:-----------------------------------------------------|
| Gateway (gw)      | gw.eduardo_gustavo_914.labredes.ifalarapiraca.local     |
| NameServer1 (ns1) | ns1.eduardo_gustavo_914.labredes.ifalarapiraca.local    |
| NameServer2 (ns2) | ns2.eduardo_gustavo_914.labredes.ifalarapiraca.local    |
| Samba-SRV.        | samba.eduardo_gustavo_914.labredes.ifalarapiraca.local  |
