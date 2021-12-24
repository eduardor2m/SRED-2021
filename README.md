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

# Implementação do Samba

- Instalar samba na máquina virtual

```bash
 $ sudo apt update
 $ sudo apt install samba
```

- Verificar se o samba está rodando

```bash
$ whereis samba
samba: /usr/sbin/samba /usr/lib/x86_64-linux-gnu/samba /etc/samba /usr/share/samba /usr/share/man/man8/samba.8.gz /usr/share/man/man7/samba.7.gz


$ sudo systemctl status smbd
● smbd.service - Samba SMB Daemon
     Loaded: loaded (/lib/systemd/system/smbd.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2021-03-22 23:07:17 UTC; 1h 26min ago
       Docs: man:smbd(8)
             man:samba(7)
             man:smb.conf(5)
    Process: 691 ExecStartPre=/usr/share/samba/update-apparmor-samba-profile (code=exited, status=0/SUCCESS)
   Main PID: 697 (smbd)
     Status: "smbd: ready to serve connections..."
      Tasks: 4 (limit: 460)
     Memory: 17.5M
     CGroup: /system.slice/smbd.service
             ├─697 /usr/sbin/smbd --foreground --no-process-group
             ├─737 /usr/sbin/smbd --foreground --no-process-group
             ├─738 /usr/sbin/smbd --foreground --no-process-group
             └─739 /usr/sbin/smbd --foreground --no-process-group

```

```bash
 $ netstat -an | grep LISTEN
```
![Screenshot_20211223_164224](https://user-images.githubusercontent.com/62352928/147303205-372b9477-7f74-4797-bd20-93856091b480.png)

- Backup do arquivo de configuração do samba.

```bash
$ sudo cp /etc/samba/smb.conf{,.backup}
$ ls -la
-rw-r--r--  1 root root 8942 Mar 22 20:55 smb.conf
-rw-r--r--  1 root root 8942 Mar 23 01:42 smb.conf.backup
$
$ sudo bash -c 'grep -v -E "^#|^;" /etc/samba/smb.conf.backup | grep . > /etc/samba/smb.conf'
```

- Criar arquivo novo somente com os comandos necessários do samba.

```bash
sudo nano /etc/samba/smb.conf
[global]
   workgroup = WORKGROUP
   netbios name = samba-srv
   security = user
   server string = %h server (Samba, Ubuntu)
   interfaces = 127.0.0.1/8 enp0s3
   bind interfaces only = yes
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d
   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes
   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes
[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700
[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
[homes]
   comment = Home Directories
   browseable = yes
   read only = no
   create mask = 0700
   directory mask = 0700
   valid users = %S
[public]
   comment = public anonymous access
   path = /samba/public
   browsable =yes
   create mask = 0660
   directory mask = 0771
   writable = yes
   guest ok = yes
   guest only = yes
   force user = nobody
   force create mode = 0777
   force directory mode = 0777
```
![Screenshot_20211223_164833](https://user-images.githubusercontent.com/62352928/147303399-052b08e6-82cc-45df-90bc-f3d35ab11461.png)

- Reiniciar o serviço smbd

```bash
 $ sudo systemctl restart smbd
```

- Modifica a pasta /samba/public para acesso a somente usuários do grupo sambashare

```rubi
[public]
   comment = public anonymous access
   path = /samba/public
   browsable =yes
   create mask = 0660
   directory mask = 0771
   writable = yes
   guest ok = no
   valid users = @sambashare
   #guest only = yes
   #force user = nobody
   #force create mode = 0777
   #force directory mode = 0777
```

- Crie um usuário do S.O para que possa utilizar o compartilhamento samba, usuário: aluno, senha: alunoifal

```rubi
$ sudo adduser aluno
Adding user `aluno' ...
Adding new group `aluno' (1001) ...
Adding new user `aluno' (1001) with group `aluno' ...
Creating home directory `/home/aluno' ...
Copying files from `/etc/skel' ...
New password: 
Retype new password: 
passwd: password updated successfully
Changing the user information for aluno
Enter the new value, or press ENTER for the default
	Full Name []: Aluno de SRED no IFAL Arapiraca
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n] y
```

- É necessário vincular o usuário do S.O. ao Serviço Samba. Repita a senha de aluno ou crie uma senha nova somente para acessar o compartilhamento de arquivo. Neste caso repetiremos a senha do usuário aluno

```rubi
$ sudo smbpasswd -a aluno
New SMB password:
Retype new SMB password:
Added user aluno.

$ sudo usermod -aG sambashare aluno

```

- O Samba já está instalado, agora precisamos criar um diretório para compartilhá-lo em rede.

```rubi
$ mkdir /home/<username>/sambashare/
$ sudo mkdir -p /samba/public
```

- Configure as permissões para que qualquer um possa acessar o compartilhamento público.

```rubi
sudo chown -R nobody:nogroup /samba/public
sudo chmod -R 0775 /samba/public
sudo chgrp sambashare /samba/public
```

- Cliente do compartilhamento:

```rubi
* Em um máquina com Windows (também pode usar linux os MacOS) digite no Winndows Explorer o endereço IP do servidor samba da seguinte forma:
**\\ip_do_maquina**. Exemplo: \\10.9.24.124
```
![Screenshot_20211223_172009](https://user-images.githubusercontent.com/62352928/147303838-a9eba405-8959-4239-adea-684847f334bd.png)

# Configuração do Bind9 (DNS Server)

## Instalação 
   * O BIND9 é a aplicação de DNS que roda no servidor.
   * Instalar o bind9 via apt-get
```bash
$ sudo apt-get install bind9 dnsutils bind9-doc 
```
   * Verifique o status do serviço:
```bash
$ sudo systemctl status bind9
```
   * Se não estiver rodando:
```bash
$ sudo systemctl enable bind9
```



## Diretórios do bind
   * Os arquivos do bind ficam na no diretório **/etc/bind**. 
```bash
$ ls /etc/bind
```
```
total 64
drwxr-sr-x  3 root bind 4096 Oct 15 09:06 .
drwxr-xr-x 94 root root 4096 Oct  8 23:29 ..
-rw-r--r--  1 root root 2761 Aug  7 14:43 bind.keys
-rw-r--r--  1 root root  237 Aug  7 14:43 db.0
-rw-r--r--  1 root root  271 Aug  7 14:43 db.127
-rw-r--r--  1 root root  237 Aug  7 14:43 db.255
-rw-r--r--  1 root root  353 Aug  7 14:43 db.empty
-rw-r--r--  1 root root  270 Aug  7 14:43 db.local
-rw-r--r--  1 root root 3171 Aug  7 14:43 db.root
-rw-r--r--  1 root bind  463 Aug  7 14:43 named.conf
-rw-r--r--  1 root bind  490 Aug  7 14:43 named.conf.default-zones
-rw-r--r--  1 root bind  468 Oct 15 05:42 named.conf.local
-rw-r--r--  1 root bind  881 Sep 19 15:08 named.conf.options
-rw-r-----  1 bind bind   77 Sep 19 14:48 rndc.key
-rw-r--r--  1 root root 1317 Aug  7 14:43 zones.rfc1918
```

### Zonas
   * As zonas são especificadas em arquivos **db**. Vamos criar um diretório para armazendar os arquivos de zonas, que sera o diretório ***/etc/bind/zones***  
```bash
$ sudo mkdir /etc/bind/zones
```

#### Criar arquivos db
   * Criar o arquivo **db** no diretório ***/etc/bind/zones***. 
   * Os arquivos **db** são bancos de dados de resolução de nomes, ou seja, quando se sabe o nome da máquina mas não se conhece o IP. Cada zona no DNS deve ter seu próprio arquivo **db**, por exemplo: a zona *meusite.com.br* terá o arquivo **db.meusite.com.br**, já a zona *outrosite.net* terá o arquivo **db.outrosite.net**. 
   * No nosso caso o domínio/zona local será labredes.ifalarapiraca.local. Assim o arquivo db será db.labredes.ifalarapiraca.local
   
##### zona direta
   * o arquivo db.labredes.ifalarapiraca.local conterá os nomes das máquinas do domínio labredes.ifalarapiraca.local
   * Para isso faremos uma cópia do arquivo /etc/bind/db.empty
```bash
$ sudo cp /etc/bind/db.empty /etc/bind/zones/db.labredes.ifalarapiraca.local 
```

##### zona reversa
   * Utilizado quando não se conhece o IP mas sabe-se o nome do host.
   * vamos criar a zona reversa a partir do arquivo /etc/bind/db.127
```bash
  $ sudo cp /etc/bind/db.127 /etc/bind/zones/db.10.9.14.rev
```

   * Assim, o arquivo **db.10.9.14.rev** conterá a zona reversa da rede 10.9.14.0. 

   
### Editar arquivos db:

   #### zona direta: db.labredes.ifalarapiraca.local
   * edite o arquivo  **db.labredes.ifalarapiraca.local** para adcionar as informações do seu domínio
      * As linhas iniciadas com **;** são comentários 
      
```bash   
    $ sudo nano db.labredes.ifalarapiraca.local 
```
---
```
;
; BIND data file for internal network
;
$ORIGIN labredes.ifalarapiraca.local.
$TTL	3h
@	IN	SOA	ns1.labredes.ifalarapiraca.local. root.labredes.ifalarapiraca.local. (
			      1		; Serial
			      3h	; Refresh
			      1h	; Retry
			      1w	; Expire
			      1h )	; Negative Cache TTL
;nameservers
@	IN	NS	ns1.labredes.ifalarapiraca.local.
@	IN	NS	ns2.labredes.ifalarapiraca.local.
;hosts
ns1.labredes.ifalarapiraca.local.	  IN	A	10.9.14.10
ns2.labredes.ifalarapiraca.local.	  IN	A	10.9.14.11
dh1.labredes.ifalarapiraca.local.	  IN	A	10.9.14.100
gw.labredes.ifalarapiraca.local.	  IN 	A	10.9.14.1          
desktophost1    CNAME     dh1                 ; CNAME é um apelido
```

---
   #### zona reversa: db.10.9.14.rev
   * edite o arquivo **db.10.9.14.rev** para adcionar as informações da zona reversa
      * As linhas iniciadas com **;** são comentários.
   
---
```
;
; BIND reverse data file of reverse zone for local area network 10.9.14.0/24
;
$TTL    604800
@       IN      SOA     labredes.ifalarapiraca.local. root.labredes.ifalarapiraca.local. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

; name servers
@      IN      NS      ns1.labredes.ifalarapiraca.local.
@      IN      NS      ns2.labredes.ifalarapiraca.local.

; PTR Records
10   IN      PTR     ns1.labredes.ifalarapiraca.local.              ; 10.9.14.10
11   IN      PTR     ns2.labredes.ifalarapiraca.local.              ; 10.9.14.11
100  IN      PTR     dh1.labredes.ifalarapiraca.local.    	    ; 10.9.14.100
1    IN      PTR     gw.labredes.ifalarapiraca.local.               ; 10.9.14.1
```
---

### Configuração do named.conf.local
   * Para ativar as zonas descritas nos arquivos **db** deve-se editar o arquivo de configuracão do bind para informar onde eles foram salvos. As zonas são adicionadas em **/etc/bind/named.conf.local**.
   
```bash
$ sudo nano /etc/bind/named.conf.local
```
---
```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "labredes.ifalarapiraca.local" {
	type master;
	file "/etc/bind/zones/db.labredes.ifalarapiraca.local";
	allow-transfer{ 10.9.14.11; };  
	allow-query{any;};
};

zone "14.9.10.in-addr.arpa" IN {
	type master;
	file "/etc/bind/zones/db.10.9.14.rev";
	allow-transfer{ 10.9.14.11; };
};
```
---

### Verificação de sintaxe 
   * Para checar a sintaxe de configuração do BIND deve-se executar o comando named-checkconf. Este scritp checa os arquivos /etc/bind/named.conf.local.*

```bash
$sudo named-checkconf
```

###  Verificar a sintaxe dos arquivos de dados
   * Para verificar se a formatação da sintaxe dos arquivos db está correta, utiliza-se o script named-checkconf da seguinte forma: ***named-check-zone <zone> <db_file>***

```bash
$ cd /etc/bind/zones
$ sudo named-checkzone labredes.ifalarapiraca.local db.labredes.ifalarapiraca.local
zone labredes.ifalarapiraca.local/IN: loaded serial 1
OK
$ sudo named-checkzone 14.9.10.in-addr.arpa db.10.9.14.rev
zone 14.9.10.in-addr.arpa/IN: loaded serial 1
OK
```

### Configure para somente resolver endereços IPv4

```bash
$sudo nano /etc/default/named
```
- adicione a linha ***OPTIONS="-4 -u bind"***
```#
# run resolvconf?
RESOLVCONF=no

# startup options for the server
OPTIONS="-4 -u bind"
```


### Execute o BIND 
```bash
$ sudo systemctl enable bind9
$ sudo systemctl restart bind9
```

### Configuração dos clientes
   * Configure o dns no nas máquina ns1, ns2 e us adicionando os campos abaixo na interface de rede local deses servidores. Observe que na máquina gw essa configuração deve ser inserida na interface de rede local (enp0s8)
```
            nameservers: 
                addresses:
                - 10.9.14.10
                - 10.9.14.11
                search: [labredes.ifalarapiraca.local]
```
   * O arquivo de configuração do netplan ficará da seguinte forma:

```bash
$ sudo nano /etc/netplan/50-cloud-init.yaml 

network:
    ethernets:
        enp0s3:                        # interface local
            addresses: [10.9.14.10/24]  # ip/mascara
            gateway4: 10.9.14.1         # ip do gateway
            dhcp4: false               # 'false' para conf. estatica 
            nameservers:               # servidores dns
                addresses:
                - 10.9.14.10            # ip do ns1
                - 10.9.14.11            # ip do ns2
                search: [labredes.ifalarapiraca.local]  # domínio
    version: 2
```

   * o campo search indica o nome do domínio no qual a máquina pertence.
   
   
---
### Testando o servidor DNS:

#### Teste de configuração como cliente. 
   * Observe se os campos **DNS servers** e **DNS Domain** estão corretos.
```bash
$ systemd-resolve --status enp0s3
```
```
Link 2 (enp0s3)
      Current Scopes: DNS
       LLMNR setting: yes
MulticastDNS setting: no
      DNSSEC setting: no
    DNSSEC supported: no
         DNS Servers: 10.9.14.10
                      10.9.14.11
         DNS Domain: labredes.ifalarapiraca.local
```
---
#### Teste o serviço DNS para a máquina ns1. 
   * Veja a resposta em **ANSWER SECTION**.
```bash
$ dig ns1.labredes.ifalarapiraca.local
```

```
; <<>> DiG 9.11.3-1ubuntu1.9-Ubuntu <<>> ns1.labredes.ifalarapiraca.local
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 47542
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;ns1.labredes.ifalarapiraca.local. IN	A

;; ANSWER SECTION:
ns1.labredes.ifalarapiraca.local. 5204 IN A	10.9.14.10

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Tue Oct 15 06:02:20 UTC 2019
;; MSG SIZE  rcvd: 7
```
---

#### Teste o serviço DNS reverso para a máquina ns1. 
```bash    
$ dig -x 10.9.14.10
```
```
; <<>> DiG 9.11.3-1ubuntu1.9-Ubuntu <<>> -x 10.9.14.10
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 48674
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;10.9.14.10.in-addr.arpa.		IN	PTR

;; ANSWER SECTION:
10.9.14.10.in-addr.arpa.	6141	IN	PTR	ns1.labredes.ifalarapiraca.local.

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Tue Oct 15 06:01:23 UTC 2019
;; MSG SIZE  rcvd: 97
```
---
#### Teste o serviço DNS reverso para a máquina ns2. 
```bash  
$ dig -x 10.9.14.11
```
```
; <<>> DiG 9.11.3-1ubuntu1.9-Ubuntu <<>> -x 10.9.14.11
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 56462
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;11.14.9.10.in-addr.arpa.		IN	PTR

;; ANSWER SECTION:
11.14.9.10.in-addr.arpa.	6177	IN	PTR	ns2.labredes.ifalarapiraca.local.

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Tue Oct 15 06:01:01 UTC 2019
;; MSG SIZE  rcvd: 97
```
