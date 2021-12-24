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

# Configurar DNS

- Acesse o arquivo 00-installer-config.yaml

```bash
 $ sudo nano /etc/netplan/00-installer-config.yaml
```

- Código para configuração estática do IP

```rubi
network:
    ethernets:
        ens160:                           # nome da interface que está sendo configurada. Verifique com o comando 'ifconfig -a'
            addresses: [10.0.0.106/24]     # IP e Máscara do Host. Aqui é só um exemplo, tenha certeza do IP do seu host, ou perderá o acesso remoto.
            gateway4: 10.0.0.1            # IP do Gateway, Aqui é só um exemplo, tenha certeza do IP do seu gateway, ou perderá o acesso remoto.
            dhcp4: false                  # dhcp4 false -> cliente DHCP está desabilitado, logo o utilizará o IP do campo 'addresses'
            nameservers:
                addresses:
                   - 8.8.8.8              # IP do servidor de nomes 1, neste caso é o IP de DNS do google
                   - 8.8.4.4              # IP do servidor de nomes 2, neste caso é outro IP de DNS do google
                search: []                # identificação do domínio, aqui neste caso está vazio.
    version: 2
```
![Screenshot_20211223_172321](https://user-images.githubusercontent.com/62352928/147302787-97cff355-d992-4a9c-a64a-bbcd61ec2e41.png)


- Para salvar as configurações use: 

```bash
 $ sudo netplan apply
```

- Para ver as configurações use:

```bash
 $ ifconfig -a
```
