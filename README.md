# Instituto Federal de Alagoas - Campus Arapiraca
### Prof. Alaelson Jatobá

### Alunos Turma 914: 
 - Bruno Martins Santos
 - Elias José Lima da Silva 
 - Filipe Alves de Souza
 - Pedro Carlos Ormindo da Silva

---
## Sumário

## 1. Introdução

## 2. Definições Iniciais

## 3. Implementação dos Serivços de Rede (Cada serviço uma sessão)

## Considerações Finais

---

# Introdução

 Nesse projeto vamos desenvolver um sistema de rede local, utilizando maquinaas virtuais com sistema operacinal Unbuto, conectados a ela via vpn.
 Aqui vamos Fazer o pass a passo de intalação e configurações de servços
 - Samba 
  > Serviço de compartilhamento de arquivos entre linux e windowns
 - Dns
  > Serviço de resolução de nomes (criação de dominio ``grupo4.turma914.ifalara.local``
 - Web
  > Serviço LAMP (Linux Apache MySQL PHP)
 - Gateway
  > Firewall de entrada as outras máquinas

# Definições Iniciais

### CONFIGURAÇÕES DAS INTEFACES DE REDE

| Rede | IP Externo | Mascara | IP Interno | Mascara |
| --- | --- | --- | --- | --- |
| IP da Subrede: | 10.9.14.0 | /24 | 192.168.14.24 | /29 |
| IP de Broadcast: | 10.9.14.255/24 | /24 | 192.168.14.31	| /29 |

| Nome da VMs | WAN | IP | LAN | IP |
| --- | --- | --- | --- | --- |
| IP do GW: | ens160 | 10.9.14.103 | ens192 |	192.168.14.25 |
| IP do SAMBA: | ens160 | 10.9.14.108 | ens192 |	192.168.14.26 |
| IP do NS1: | ens160 | 10.9.14.126 | ens192 |	192.168.14.27 |
| IP do NS2: | ens160 | 10.9.14.109 | ens192 |	192.168.14.28 |
| IP do WEB | ens160 | 10.9.14.217 | ens192 |	192.168.14.29 |
| IP do BD | ens160 | 10.9.14.218 | ens192 |	192.168.14.30 |

### DEFINIÇÂO DE NOMES E DOMÍNIO (grupo4.turma914.ifalara.local):				

| VM | Nome | Domínio (zona): | grupo4.turma914.ifalara.local |
| --- | --- |--- | --- |
| Aluno08 | Bruno | FQDN do GW: | gw.grupo4.turma914.ifalara.local |
| Aluno03 | Elias |	FQDN do SAMBA: | smb.grupo4.turma914.ifalara.local |
| Aluno26	| Pedro | FQDN do NS1: | ns1.grupo4.turma914.ifalara.local |
| Aluno09 | Filipe | FQDN do NS2: | ns2.grupo4.turma914.ifalara.local |	
| Grupo4vm1	| Vm1 | FQDN do WEB	| www.grupo4.turma914.ifalara.local |	
| Grupo4vm2 | Vm2 |	FQDN do BD | bd.grupo4.turma914.ifalara.local |

# Implementação dos Serivços de Rede (Cada serviço uma sessão)

## Samba

Vamos condigurar a maquina "smb" nela vamosadiconar a resolução de nomes e o serviço de compartilhamento de arquivos SAMBA.

### 0 - Primeiro vamos mudar o nome da maquina para o nome escolhhido que está na tabela:

  > smb.grupo4.turma914.ifalara.local
  Com o comando:

  ```
  $ sudo hostnamectl set-hostname smb.grupo4.turma914.ifalara.local
  ```
  ---

### 1 - Primeiro vamos configurar essa maquina como cliente do serviço DNS. Para isso vamos configurar as interfaces de rede e adicionar os IPs de DNS e o compo "search".

  Vamos entrar no arquivo de configuração:

  ```
  $ sudo nano /etc/netplan/00-installer-config.yaml
  ```

  **RESULTADO**

  ```
  #This is the network config written by 'subiquity'
  network:
      ethernets:
          ens160:
            dhcp4: false
            addresses: [10.9.14.108/24]
            gateway4: 10.9.14.1
            nameservers:
              addresses:
                 - 10.9.14.126
                 - 10.9.14.109
              search: [grupo4.turma914.ifalara.local]
          ens192:
            dhcp4: false
            addresses: [192.168.14.26/29]
      version: 2
  ```
  ![install_confing](/smb/install_confing.PNG)

  Em seguida vamos aplicar as alterações.

  Para isso usamos:

  ```
  $ sudo netplan apply
  ```
  ---

### 2 - Agora vamos testar se está funcionando normalmente.

  ```
  $ dig @10.9.14.126 gw.grupo4.turma914.ifalara.local
  ```

  Na linha "ANSWER SECTION:" tem que aparecer que foi resolvido, com o dominio e o IP.

  Vamos usar o seguinte comando para verificar se a interface 160 está funcionando:

  ```
  $ systemd-resolve --status ens160
  ```

  Para finalizar vamos ver se o nosso serviço DNS revolve o DNS do google:

  ```
  $ ping google.com
  ```

  ![dig-system_resolve-ping](/smb/dig-system_resolve-ping.PNG)

  ---

### 3 - Vamos instalar o Samba

  Sempre antes de instalar qualquer coisa e importante dar um update na maquina, então usamos:


  ```
  $ sudo apt update
  ```
  ```
  $ sudo apt install samba 
  ```

  Para ver o status usamos:

  ```
  $ sudo systemctl status smbd
  ```

  As portas TCP 445 e 136 estão rodando na nossa maquina 

  ```
  $ netstat -an | grep LISTEN
  ```
  ```
  tcp        0      0 0.0.0.0:445             0.0.0.0:*               LISTEN     
  tcp        0      0 0.0.0.0:139             0.0.0.0:*               LISTEN   
  ```

  ![grep_listen_antes](/smb/grep_listen_antes.PNG)

  Vamos fazer o backup do arquivo de configuração do samba e cria um arquivo novo

  ```
  $ sudo cp /etc/samba/smb.conf{,.backup}
  ```

  Para verificar se foi criado o novo arquivo

  ```
  $ ls -la /etc/samba
  ```

  **RESULTADO**
  ```
  -rw-r--r--  1 root root 8942 Mar 22 20:55 smb.conf
  -rw-r--r--  1 root root 8942 Mar 23 01:42 smb.conf.backup
  ```

  Agora usamos o comando grep para remover os comentarios dos arquivos

  ```
  $ sudo bash -c 'grep -v -E "^#|^;" /etc/samba/smb.conf.backup | grep . > /etc/samba/smb.conf'
  ```

  Agora vamos acessar e editar o arquivo de configurações do samba

  ```
  $ sudo nano /etc/samba/smb.conf
  ```

  Deixando ele da seguinte maneira

  **RESULTADO**
  ```
  [global]
     workgroup = WORKGROUP
     netbios name = samba-srv
     security = user
     server string = %h server (Samba, Ubuntu)
     interfaces = 127.0.0.1/8 ens160 ens192
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

  Vamos modiificar a pasta public para que apenas usuarios que participem do grupo sambashare possam acessar

  ```
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

  ![smb_conf](/smb/smb_conf.PNG)

  Após salvar vamos reiniciar o nosso servidor

  ```
  $ sudo systemctl restart smbd
  ```

  Agora podemos verificar que as portas 445 e 139 estão funcionando nas interfaces ens160 e ens192

  ```
  $ netstat -an | grep LISTEN
  ```

  **RESULTADO**
  ```
  tcp        0      0 10.9.14.108:445             0.0.0.0:*               LISTEN     
  tcp        0      0 10.9.14.108:139             0.0.0.0:*               LISTEN  
  tcp        0      0 192.168.14.126:445             0.0.0.0:*            LISTEN     
  tcp        0      0 192.168.14.126:139             0.0.0.0:*            LISTEN   
  ```

  ![grep_listen_depois](/smb/grep_listen_depois.PNG)

  Vamos criar um usuario: ``aluno`` com a senha: ``alunoifal``

  ```
  $ sudo adduser aluno
  ```
  ```
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

  Para vincular o novo usuário ao serviço samba usamos:

  Caso queira mudar a senha:

  ```
  $ sudo smbpasswd -a aluno
  ```
  ```
  New SMB password:
  Retype new SMB password:
  Added user aluno.
  ```

  Agora vamos adicionalo ao grupo sambashare

  ```
  $ sudo usermod -aG sambashare aluno
  ```

  Vamos criar um diretorio com compartilhamento em rede

  ```
  $ sudo mkdir /home/aluno/sambashare
  ```
  ```
  $ sudo mkdir -p /samba/public
  ```

  Agora vamos trocar a permição do pasta public

  ```
  $ sudo chown -R nobody:nogroup /samba/public
  ```
  ```
  $ sudo chmod -R 0775 /samba/public
  ```
  ```
  $ sudo chgrp sambashare /samba/public
  ```

  ###  - Vamos testar

  Para testar vamos no nosso explorador de arquivos

  Em rede digitamos na barra de rotas o nosso IP:

  ```
  \\10.9.14.108
  ```

  ![teste_smb_ip](/smb/teste_smb_ip.PNG)

  Enstão aparecerá as nossas pastas

  Já que nosso DNS já está implementado podemos acessar usando o nome da maquina ``smb`` de IP: ``10.9.14.108``

  ```
  \\smb.grupo4.turma914.ifalara.local
  ```

  ![teste_smb_nome](/smb/teste_smb_nome.PNG)

  Para acessar a pasta public é nescessario usar nosso usuario ``aluno`` e a senha: ``alunoifal``

  Vamos criar um arquivo no explorador de arquivos do windows para testar


  ![teste_smb_texto](/smb/teste_smb_texto.PNG)

![teste_smb_ls](/smb/teste_smb_ls.PNG)


#### Nosso serviço samba está funcionando!


## DNS 

### Configuração do Bind9 (DNS Server)

#### Master

  Nesta maquina ns1 vamos configurar o serviço de resolução de nome (DNS) com o Bind9, utilizando à como DNS Master. 

### 0 - Antes de tudo vamos mudar o nome da maquina para o nome escolhido que está na tabela:

  > ns1.grupo4.turma914.ifalara.local
  
  Com o comando:

  ```
  $ sudo hostnamectl set-hostname ns1.grupo4.turma914.ifalara.local
  ```

  Para que a alteração seja mostrada é nescessario dar u reboot na maquina

  ```
  $ Sudo reoobt
  ```
  
  ![hostname-reboot](/ns1/hostname-reboot.PNG)

  Para testar se o nome funcionou basta se conectar novamente com a maquina virtual e usar: 
  
  ```
  $ hostname
  ```

  ![teste_hostname-ip](/ns1/teste_hostname-ip.PNG)
  
---


### 1 - Como primeiro passo devemos instalar o bind9.

  Mas primeiro vamos fazer um update:
  
  ```
  $ sudo apt update
  ```
  
  Agora vamos instalar o bind:
  
  ```
  $ sudo apt-get install bind9 dnsutils bind9-doc
  ```
  
  Em seguida vamos verificar se o Bind9 está funcionando:
  
  ```
  $ sudo systemctl status bind9
  ```

  ![status_bind](/ns1/status_bind.PNG)

---

### 2 - Vamos configurar os arquivos de zonas.

  Antes vamos criar o diretorio zones para alocar os nossos arquivos de zonas e seguir o padrão, com: 
  
  ```
  $ sudo mkdir /etc/bind/zones
  ```

  Agora vamos copiar os arquivos para a pasta zones.

##### Zona Direta:

  ```
  $ sudo cp /etc/bind/db.empty /etc/bind/zones/db.grupo4.turma914.ifalara.local
  ```

  > Obs.: grupo4.turma914.ifalara.local é o nome do dominio escolhido pelo grupo 4;

##### Zona Reversa:
  ```
  $  sudo cp /etc/bind/db.127 /etc/bind/zones/db.10.9.14.rev
  ```
  > Obs.: 10.9.14 é a nossa rede;

  Vamos editar o arquivo "db.grupo4.turma914.ifalara.local":
  ```
  $ sudo nano /etc/bind/zones/db.grupo4.turma914.ifalara.local
  ```
  Nele vamos colocar os DNS e os IPs:

**RESULTADO**
  ```
  ;
  ; BIND data file for internal network
  ;
  $ORIGIN grupo4.turma914.ifalara.local.
  $TTL	3h
  @	IN	SOA	ns1.grupo4.turma914.ifalara.local. root.grupo4.turma914.ifalara.local. (
              2022030900		; Serial
              3h	; Refresh
              1h	; Retry
              1w	; Expire
              1h )	; Negative Cache TTL
  ;nameservers
  @	IN	NS	ns1.grupo4.turma914.ifalara.local.
  @	IN	NS	ns2.grupo4.turma914.ifalara.local.

  ;hosts
  ns1.grupo4.turma914.ifalara.local.	  IN	A	10.9.14.126
  ns1.grupo4.turma914.ifalara.local.	  IN	A	10.9.14.109
  smb.grupo4.turma914.ifalara.local.	  IN	A	10.9.14.108
  gw.grupo4.turma914.ifalara.local.	  IN 	A	10.9.14.103
  www.grupo4.turma914.ifalara.local.	  IN 	A	10.9.14.217
  bd.grupo4.turma914.ifalara.local.	  IN 	A	10.9.14.218
  ```
  
  ![zonaDireta](/ns1/zonaDireta.PNG)
  
  ---

  Vamos editar o arquivo "db.10.9.14.rev":
  
  ```
  $ sudo nano /etc/bind/zones/db.10.9.14.rev
  ```
  
  Nele vamos colocar os DNS e os IPs:

**RESULTADO**
  ```
  ;
  ; BIND reverse data file of reverse zone for local area network 10.9.14.0/24
  ;
  $TTL    604800
  @       IN      SOA     grupo4.turma914.ifalara.local. root.grupo4.turma914.ifalara.local. (
                                2022030900         ; Serial
                           604800         ; Refresh
                            86400         ; Retry
                          2419200         ; Expire
                           604800 )       ; Negative Cache TTL

  ; name servers
  @      IN      NS      ns1.grupo4.turma914.ifalara.local.
  @      IN      NS      ns2.grupo4.turma914.ifalara.local.

  ; PTR Records
  126   IN      PTR     ns1.grupo4.turma914.ifalara.local.
  109   IN      PTR     ns2.grupo4.turma914.ifalara.local.
  103   IN      PTR     gw.grupo4.turma914.ifalara.local.
  108   IN      PTR     smb.grupo4.turma914.ifalara.local.
  217   IN      PTR     www.grupo4.turma914.ifalara.local.
  218   IN      PTR     bd.grupo4.turma914.ifalara.local.
  ```

  ![zonaReversa](/ns1/zonaReversa.PNG)

---

### 3 - Vamos ativar as zonas, então devesse configurar o arquivo "named.conf.local":

  ```
  $  sudo nano /etc/bind/named.conf.local
  ```

**RESULTADO**
  ```
  //
  // Do any local configuration here
  //

  // Consider adding the 1918 zones here, if they are not used in your
  // organization
  //include "/etc/bind/zones.rfc1918";

  Nele vamos adicionas as seguintes linhas de codigos logo a baixo, pulando uma linha:

  zone "grupo4.turma914.ifalara.local" {
    type master;
    file "/etc/bind/zones/db.grupo4.turma914.ifalara.local";
    allow-transfer{ 10.9.14.109; };  
    allow-query{any;};
  };

  zone "14.9.10.in-addr.arpa" IN {
    type master;
    file "/etc/bind/zones/db.10.9.14.rev";
    allow-transfer{ 10.9.14.109; };
  };
  ```

  Em zones tem que ter o nome de dominio escolhido. Em "allow-transfer" o IP é o do ns2, ou seja Slave.

  ![namedConfLocal](/ns1/namedConfLocal.PNG)

---

### 4 - Agora vamos verificar a sintax dos arquivos que editamos.

  Para checar a sintax do arquivo named.conf.local usamos:
  
  ```
  $ sudo named-checkconf
  ```
  
  Vamos entrar no diretório zones:
  
  ```
  $ cd /etc/bind/zones
  ```
  
  Vamos Verificar se o arquivo "db.grupo4.turma914.ifalara.local" está ok.
  
  ```
  $ sudo named-checkzone grupo4.turma914.ifalara.local db.grupo4.turma914.ifalara.local
  ```

**RESULTADO**
  ```
  ...
  zone grupo4.turma914.ifalara.local/IN: loaded serial 1
  OK
  ```

  Vamos Verificar se o arquivo "db.10.9.14.rev" está ok.
  
  ```
  $ sudo named-checkzone 14.9.10.in-addr.arpa db.10.9.14.rev
  ```

**RESULTADO**

  ```
  ...	
  zone 14.9.10.in-addr.arpa/IN: loaded serial 1
  OK
  ```

---

### 5 - Vamos configurar para resolver apenas IPv4.

  Entraremos no arquivo name e adicionaremos "-4" na linha "OPTINS=-u bind":
  
  ```
  $ sudo nano /etc/default/named
  ```
  
**RESULTADO**

  ```
  # run resolvconf?
  RESOLVCONF=no

  # startup options for the server
  OPTIONS="-4 -u bind"
  ```
---

### 6 - Vamos reiniciar o bind, com:

  ```
  $ sudo systemctl restart bind9
  ```
---

### 7- Agora vamos configurar as nossas interfaces, ens160 e ens192:

  ```
  $ sudo nano /etc/netplan/00-installer-config.yaml
  ```

  Na interface ens160 vamos retirar os endereços de IPs e adiconaremos os indereçoes do Master e Sleve.
**RESULTADO**
  ```
  network:
    ethernets:
      ens160:
        addresses: [10.9.14.126/24]
        gateway4: 10.9.14.1
        dhcp4: false
        nameservers:
          addresses:
            - 10.9.14.126			# ip do ns1
            - 10.9.14.109			# ip do ns2
          search: [grupo4.turma914.ifalara.local] # dominio
      ens192:
        addresses: [192.168.14.27/29] # ip configurado de acordo com a tabela
    version: 2
  ```
  
  ![00_installer_config](/ns1/00_installer_config.PNG)

---

### 8 - Agora vamos testar o DNS Marter.
  
  ```
  $ dig @10.9.14.126 gw.grupo4.turma914.ifalara.local
  ```
  
  Na linha "ANSWER SECTION:" tem que aparecer que foi resolvido, com o dominio e o IP.

  Vamos usar o seguinte comando para verificar se a interface 160 está funcionando:
  
  ```
  $ systemd-resolve --status ens160
  ```
  
  Para finalizar vamos ver se o nosso serviço DNS revolve o DNS do google:
  
  ```
  $ ping google.com
  ```
  
  ![dig-system_resolve-ping](/ns1/dig-system_resolve-ping.PNG)

---

#### Slave

##### Agora vamos configurar o Dns Slave com os dominios corretos na maquina ns2.

### 0 - Primeiro vamos corrigir o nome da maquina, colocando de acordo com a planilha de acompanhamento;
  
  ```
  $ sudo hostnamectl set-hostname ns2.grupo4.turma914.ifalara.local
  ```

### 1 - Como primeiro devemos instalar o bind9.
  Mas primeiro vamos fazer um update:
  
  ```
  $ sudo apt update
  ```
  
  Agora vamos instalar o bind:
  
  ```
  $ sudo apt-get install bind9 dnsutils bind9-doc
  ```
  
  Em seguida vamos verificar se o Bind9 está funcionando:
  
  ```
  $ sudo systemctl status bind9
  ```

---

### 2 - Vamos logo confugurar as interfaces de redes. Usamos:

  ```
  $ sudo nano /etc/netplan/00-installer-config.yaml
  ```

**RESULTADO**
  ```
  # This is the network config written by 'subiquity'
  network:
    ethernets:
      ens160:
        addresses: [10.9.14.109/24]
        gateway4: 10.9.14.1
        dhcp4: false
        nameservers:
          addresses:
            - 10.9.14.126         # IP ns1 (pedro)
            - 10.9.14.109         # IP ns2 (filipe)
          search: [grupo4.turma914.ifalara.local]
      ens192:
        addresses: [192.168.14.28/29]
        nameservers:
          addresses:
            - 192.168.14.25
            - 192.168.14.26
          search: [grupo4.turma914.ifalara.local]
    version: 2
  ```

  ![00-installer_config](/ns2/00-installer_config.PNG)

  Para aplicar usamos:
  
  ```
  $ sudo netplan apply
  ```
  
  Em seguida vamos verificar se está ok, com:
  
  ```
  $ ifconfig
  ```
  
  ![ifconfig](/ns2/ifconfig.PNG)
  
---

### 3 - Agora verificamos se o Bind esta rodando com:

  ```
  $ sudo systemctl status bind9
  ```
  
  ![status_bind](/ns2/status_bind.PNG)

  Agora vamos editar o o arquivo "named.conf.local" informando que esse é o DNS Slave e qual o ip e o arquivo de DNS do Master. 
  
  Usamos o comando:
  
  ```
  $ sudo nano /etc/bind/named.conf.local
  ```

**RESULTADO**

  ```
  //
  // Do any local configuration here
  //

  // Consider adding the 1918 zones here, if they are not used in your
  // organization
  //include "/etc/bind/zones.rfc1918";

  zone "grupo4.turma914.ifalara.local" {
          type slave;
          file "/etc/bind/zones/db.ifalara.local";
          masters { 10.9.14.126; };
  };

  zone "14.9.10.in-addr.arpa" IN {
          type slave;
          file "/etc/bind/zones/db.10.9.14.rev";
          masters { 10.9.14.126; };
  };
  ```
  
  ![named_conf_local](/ns2/named_conf_local.PNG)

  Par verificar a sintax usamos:
  
  ```
  $ sudo named-checkconf
  ```
  
  Se não apareceu nem um erro então está ok.

---

### 4 - Agora Vamos testar para ver se está funcionando, com:

```
$ systemd-resolve --status
```

E por fim testarmos se o nosso DNS está rtesolvendo o nome do google.

```
$ ping google.com
```

![systemd_resolve-ping](/ns2/systemd_resolve-ping.PNG)

--

## Página Web

  Nesta maquina "www" vamos configurar o servidor web usando a ferramenta Apache2 e configurar a maquina como cliente do serviço DNS. 

### 0 - Primeiro vamos mudar o nome da maquina para o nome escolhhido que está na tabela:

  > www.grupo4.turma914.ifalara.local

  Com o comando:

  ```
  $ sudo hostnamectl set-hostname ns2.grupo4.turma914.ifalara.local
  ```
---

### 1 - Primeiro vamos configurar essa maquina como cliente do serviço DNS. Para isso vamos configurar as interfaces de rede e adicionar os IPs de DNS e o compo "search".

  Vamos entrar no arquivo de configuração:

  ```
  $ sudo nano /etc/netplan/00-installer-config.yaml
  ```

**RESULTADO**
  ```
  #This is the network config written by 'subiquity'
  network:
      ethernets:
          ens160:
            dhcp4: false
            addresses: [10.9.14.217/24]
            gateway4: 10.9.14.1
            nameservers:
              addresses:
                 - 10.9.14.126
                 - 10.9.14.109
              search: [grupo4.turma914.ifalara.local]
          ens192:
            dhcp4: false
            addresses: [192.168.0.29/29]
      version: 2
  ```
  
  ![installer_confing](/www/installer_confing.PNG)

  Em seguida vamos aplicar as alterações.
  
  > Para isso usamos:

  ```
  $ sudo netplan apply
  ```

---

### 2 - Agora vamos testar se está funcionando normalmente.

  ```
  $ dig @10.9.14.126 gw.grupo4.turma914.ifalara.local
  ```
  
  Na linha "ANSWER SECTION:" tem que aparecer que foi resolvido, com o dominio e o IP.

  Vamos usar o seguinte comando para verificar se a interface 160 está funcionando:
  
  ```
  $ systemd-resolve --status ens160
  ```

  Para finalizar vamos ver se o nosso serviço DNS revolve o DNS do google:
  
  ```
  $ ping google.com
  ```
  
  ![dig-system_resolve-ping](/www/dig-system_resolve-ping.PNG)

---

### 3 - Agora vomos instalar o apache e configurar para subir nosso servidor apacha.

  Primeiro vamos fazer um update na nossa maquina, via apt:
  
  ```
  $ sudo apt update
  ```

  Para instalar o apache2, versão que vamosusar, usamos:

  ```
  $ sudo apt install apache2
  ```
  
  Obs.: Não é necessário configurar o firewall, pois isso vai ser feito na maquina "gw".
  
---

### 4 - Vamos verificar se o servidor web está funcionando, usamos:

  ```
  $ sudo systemctl status apache2
  ```

**RESULTADO**
  ```
  ● apache2.service - The Apache HTTP Server
       Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
       Active: active (running) since Tue 2022-03-08 12:30:50 UTC; 4 days ago
         Docs: https://httpd.apache.org/docs/2.4/
     Main PID: 3630 (apache2)
        Tasks: 55 (limit: 1066)
       Memory: 7.0M
       CGroup: /system.slice/apache2.service
               ├─  3630 /usr/sbin/apache2 -k start
               ├─129424 /usr/sbin/apache2 -k start
               └─129425 /usr/sbin/apache2 -k start

  Mar 08 12:30:50 www systemd[1]: Started The Apache HTTP Server.
  Mar 10 00:00:29 www systemd[1]: Reloading The Apache HTTP Server.
  Mar 10 00:00:29 www apachectl[51821]: AH00558: apache2: Could not reliably determine the serv>
  Mar 10 00:00:29 www systemd[1]: Reloaded The Apache HTTP Server.
  Mar 11 00:00:01 www systemd[1]: Reloading The Apache HTTP Server.
  Mar 11 00:00:01 www apachectl[95559]: AH00558: apache2: Could not reliably determine the serv>
  Mar 11 00:00:01 www systemd[1]: Reloaded The Apache HTTP Server.
  Mar 12 00:00:37 www systemd[1]: Reloading The Apache HTTP Server.
  Mar 12 00:00:37 www apachectl[129422]: AH00558: apache2: Could not reliably determine the ser>
  Mar 12 00:00:37 www systemd[1]: Reloaded The Apache HTTP Server.
  ```

  ![status_apache2](/www/status_apache2.PNG)

  Para acessar o servidor web basta ir no navegador e colocar o IP da maquina "www", onde foi feito a instalação do Aopache2.
  Caso não saiba seu IP basta digitar:
  
  ```
  $ hostname -i
  ```
  
  Agora é só ir ao navegador e digitar o IP. Se o servidor web estiver rodando aparecerá uma pagina do Apache2.
  
---

### 5 - Agora vamos criar o nosso site.

  Primeiro criarmos um diretorio com o nome do nosso domínio, para isso usamos:
  
  ```
  $ sudo mkdir /var/www/www.grupo4.turma914.ifalara.local
  ```
  
  Agora atribuimos a propriedade do diretório com a variável de ambiente $USER:
  
  ```
  $ sudo chown -R $USER:$USER /var/www/www.grupo4.turma914.ifalara.local
  ```
  
  Agora vamos configurar as permições dos webs hosts, com:
  
  ```
  $ sudo chmod -R 755 /var/www/www.grupo4.turma914.ifalara.local
  ```
  
  Agora vomos apenas adicionar os scripts de testes disponibilizados pelo professor
  
  ```
  $ ls -la /var/www/www.grupo7.turma914.ifalara.local
  ```
  ```
  drwxr-xr-x 3 administrador administrador 4096 Mar 17 13:33 .
  drwxr-xr-x 4 root          root          4096 Mar 17 13:25 ..
  -rw-rw-r-- 1 administrador administrador 3922 Mar 12 06:56 consult.php
  drwxrwxr-x 2 administrador administrador 4096 Mar 17 13:30 css
  -rw-rw-r-- 1 administrador administrador 1233 Mar 12 07:44 deleteAluno.php
  -rw-rw-r-- 1 administrador administrador 1488 Mar 12 12:56 deleteGrupo.php
  -rw-rw-r-- 1 administrador administrador 1227 Mar 12 07:20 deleteHost.php
  -rw-rw-r-- 1 administrador administrador 1292 Mar 12 14:35 delete.html
  -rw-rw-r-- 1 administrador administrador 1430 Mar 12 13:40 index.html
  -rw-rw-r-- 1 administrador administrador   20 Mar  9 01:56 info.php
  -rw-rw-r-- 1 administrador administrador 1105 Mar 12 14:33 insertAluno.php
  -rw-rw-r-- 1 administrador administrador 1114 Mar 12 14:32 insertGrupo.php
  -rw-rw-r-- 1 administrador administrador 1112 Mar 12 14:32 insertHost.php
  -rw-rw-r-- 1 administrador administrador 1282 Mar 12 14:33 insert.html
  -rw-rw-r-- 1 administrador administrador  606 Mar 11 18:35 insert.php
  -rw-rw-r-- 1 administrador administrador 1180 Mar 12 14:19 updateAluno.php
  -rw-rw-r-- 1 administrador administrador 1168 Mar 12 14:25 updateGrupo.php
  -rw-rw-r-- 1 administrador administrador 1186 Mar 12 14:30 updateHost.php
  -rw-rw-r-- 1 administrador administrador 1594 Mar 12 14:16 update.html
  -rw-rw-r-- 1 administrador administrador 3922 Mar 12 06:56 update.php
  ```
  
  Para que o Apache apresente a nossa página vamos copiar um arquivo de configuração:
  
  ```
  $ sudo nano /etc/apache2/sites-available/www.grupo7.turma914.ifalara.local.conf
  ```
  
  Neste arquivo vamos adiciopnar o seguinte bloco de código:
  
  ```
  <VirtualHost *:80>
   ServerAdmin webmaster@localhost
          DocumentRoot /var/www/www.grupo4.turma914.ifalara.local
          ServerAlias www.grupo4.turma914.ifalara.local
          DocumentRoot /var/www/www.grupo4.turma914.ifalara.local
  </VirtualHost>
  ```
  
  ![domain_conf](/www/domain_conf.PNG)

  Em segida vamos habilitar o arquivo com a ferramenta a2ensite:
  
  ```
  $ sudo a2ensite www.grupo4.turma914.ifalara.local.conf
  ```
  
  Vamos desabilitar o site padrão definido em 000-default.conf:
  
  ```
  $ sudo a2dissite 000-default.conf
  ```
  
  Em seguida, vamos testar à procura de erros de configuração:
  
  ```
  $ sudo apache2ctl configtest
  ```

  Vamos modificar as permissões de dono da pasta "DocumentRoot" para "www-data", que é o usuário do apache.

  ```
  $ chown -R www-data:www-data www.grupo4.turma914.ifalara.local
  ```

  Vamos reiniciar o Apache com:

  ```
  $ sudo systemctl restart apache2
  ```

  Agora é só acessar novamente no navegador com o IP da maquina. 
  
  ![a2ensite-configtest-restart](/www/a2ensite-configtest-restart.PNG)
  
  ![site](/www/site.PNG)

--

## php

  Ainda na maquina "www" vamos fazer a instalação do php

  Vamos fazer um update na maquina:

  ```
  $ sudo apt update -y
  ```

  Caso precise faça o upgrade na maquina:

  ```
  $ sudo apt upgrade -y
  ```

### 1 - Vamos isntalar o php

  ```
  $ sudo apt install php7.4 libapache2-mod-php7.4 php7.4-mysql php-common php7.4-cli php7.4-common php7.4-common php7.4-json php7.4-opcache php7.4-readline
  ```

  Para carregar o php7.4 usamos:
  ```
  $ sudo a2enmod php7.4
  ```

  Agora reioniciamos o apache2

  ```
  $ sudo systemctl restart apache2
  ```

  > Obs.: O php já está funcionandop no servidor web
  
--

Na maquina "bd" vamos configurar o servidor de banco de dados usando a ferramenta MySQL e configurar a maquina como cliente do serviço DNS. 

### 0 - Primeiro vamos mudar o nome da maquina para o nome escolhhido que está na tabela:

  > bd.grupo4.turma914.ifalara.local

Com o comando:

```
$ sudo hostnamectl set-hostname bd.grupo4.turma914.ifalara.local
```

![/bd/hostname](/bd/hostname.PNG)

1- Primeiro vamos configurar essa maquina como cliente do serviço DNS. Para isso vamos configurar as interfaces de rede e adicionar os IPs de DNS e o compo "search".
Vamos entrar no arquivo de configuração:
$ sudo nano /etc/netplan/00-installer-config.yaml
RESULTADO
#This is the network config written by 'subiquity'
network:
    ethernets:
        ens160:
          dhcp4: false
          addresses: [10.9.14.217/24]
          gateway4: 10.9.14.1
          nameservers:
            addresses:
               - 10.9.14.126
               - 10.9.14.109
            search: [grupo4.turma914.ifalara.local]
        ens192:
          dhcp4: false
          addresses: [192.168.0.29/29]
    version: 2


![/bd/hostname](/bd/installer_confing.PNG)
-

Em seguida vamos aplicar as alterações.

Para isso usamos:
```
$ sudo netplan apply
```

### 2 - Agora vamos testar se está funcionando normalmente.

```
$ dig @10.9.14.126 gw.grupo4.turma914.ifalara.local
```

Na linha "ANSWER SECTION:" tem que aparecer que foi resolvido, com o dominio e o IP.

Vamos usar o seguinte comando para verificar se a interface 160 está funcionando:

```
$ systemd-resolve --status ens160
```

Para finalizar vamos ver se o nosso serviço DNS revolve o DNS do google:

```
$ ping google.com
```

![/bd/dig-system_resolve-ping](/bd/dig-system_resolve-ping.PNG)
---

### 3 - Instalação do Mysql Server

Vamos fazer um update na maquina:

```
$ sudo apt update -y
```

Caso precise faça o upgrade na maquina:

```
$ sudo apt upgrade -y
```

### 1 - Vamos isntalar o Mysql

```
$ sudo apt install mysql-server
```

Para verificar se o serviço está funcionando basta digitar:

```
$ systemctl status mysql
```

### 2 - Agoma Vamos configurar

Vamso verificar quais as portas que estão sendoutilizadas e em quais IPs

Para isso usamos:

```
$  netstat -an | grep LISTEN
```

![/bd/netstat_antes](/bd/netstat_antes.PNG)

> Obs.: As portas "3306" e "33060" estão recebendo somente conexão da interface "127.0.0.1"

Vamos libera as portas 

```
$ sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Nas linhas "mysqlx-bind-addresse" e "bind-address" vamos substituir "127.0.0.1" por "0.0.0.0"

```
# localhost which is more compatible and is not less secure.
bind-address            = 0.0.0.0
mysqlx-bind-address     = 0.0.0.0
```

![mysqld_cnf](/bd/mysqld_cnf.PNG)

Após isso vamos reiniciar o serviço com:

```
$ systemctl restart mysql
```

Se verificamos novamente as porta agora estão recebendo IPs diferentes

```
$ netstat -an | grep LISTEN
```

![/bd/netstat_depois](/bd/netstat_depois.PNG)

### 3 - Configuração de segurança

Vamos adicionar umasenha para o MySQL, para isso usamos:

```
$ sudo mysql_secure_installation
```

```
Securing the MySQL server deployment.

Connecting to MySQL using a blank password.

VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?

Press y|Y for Yes, any other key for No:
```

Digite "y" aceitar creiar uma senha

```
Press y|Y for Yes, any other key for No: y

There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary                  file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 
```

> Obs.: Nossa senha é: ``S3nh4@ifal``

```
Please set the password for root here.

New password: 

Re-enter new password: 

Estimated strength of the password: 100 
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Success.

By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : n

 ... skipping.
Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done! 
```

Vamos iniciar o MySQL

```
$ sudo mysql
```

```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.28-0ubuntu0.20.04.3 (Ubuntu)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

Já estamos conectados no MySQL, para sair usamos: ``exit``

### 4 - Trabalhando com MySQL

Agora vamos criar o Banco de Dados ``projetofinal_sred`` com as seguintes tabelas ``grupo``, ``aluno`` e ``host``

Para isso usamos os seguintes comandos

Para criar o banco de dados:

```
CREATE DATABASE projetofinal_sred;
```

Para usar o banco de dados criado

```
USE projetofinal_sred;
```

Agora vamos criar as tabelas:

```
CREATE TABLE aluno (UID int(10) unsigned NOT NULL AUTO_INCREMENT, Nome varchar (255), Email varchar (255), GID int,PRIMARY KEY (`UID`)) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
```
CREATE TABLE grupo (GID int(10) unsigned NOT NULL, Nome varchar (255), Dominio varchar (255), PRIMARY KEY (`GID`)) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
```
CREATE TABLE host (HID int(10) unsigned NOT NULL AUTO_INCREMENT, VmName varchar (255), FQDName varchar (255), GID int, PRIMARY KEY (`HID`)) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

Para ver as tabelas basta usar:

```
SHOW TABLE;
```

Para adicionar dados nas nossas tabelas usamos:

```
INSERT INTO aluno
VALUES (value1, value2, value3, ...);
```

Caso tenha dados nas tabelas podemos ver esses dados usando:

```
SELECT * FROM aluno;
```
```
SELECT * FROM grupo;
```
```
SELECT * FROM host;
```

Vamos criar um usuário que acessa pelo localhost com: user: 'pfadmin' e senha '4dm1n@BD'

```
CREATE USER 'pfadmin'@'localhost' IDENTIFIED BY '4dm1n@BD';
```

Agora vamos dar privilegios de acesso ao usuario criado

```
GRANT ALL PRIVILEGES ON *.* TO 'pfadmin'@'localhost' WITH GRANT OPTION;
```

Vamos criar um usuário que acessa pela web com: user: 'pfadmin' e senha '4dm1n@BD'

```
CREATE USER 'pfadmin'@'10.9.14.217' IDENTIFIED BY '4dm1n@BD';
```

Agora vamos conceder privilégios de acesso a todas as tabelas

```
GRANT ALL PRIVILEGES ON *.* TO 'pfadmin'@'10.9.14.217' WITH GRANT OPTION;
```

Para sair do MySQL usamos:

```
exit
```

### 5 - Para testar o nosso banco de dados

> Obs.: Vamos usar os scripts disponibilizados pelo professor Alaelson.

Como nossa implenetação já está funcionando então vamos fazer os testes dirtetamente no site 

Já que o nosso DNS está funcionado podemos acessar o nosso site pela nome: ``www.grupo4.turma914.ifalara.local``

![/www/site](/www/site.PNG)

![/www/siteINSERT](/www/siteINSERT.PNG)

![/www/siteSELECTgrupo](/www/siteSELECTgrupo.PNG)

![/www/siteSELECTaluno](/www/siteSELECTaluno.PNG)

---

## Gateway

Vamos condigurar a maquina "smb" nela vamosadiconar a resolução de nomes e o serviço de compartilhamento de arquivos SAMBA.

### 0 - Primeiro vamos mudar o nome da maquina para o nome escolhhido que está na tabela:

> gw.grupo4.turma914.ifalara.local

Com o comando:

```
$ sudo hostnamectl set-hostname gw.grupo4.turma914.ifalara.local
```

---

### 1 - Primeiro vamos configurar essa maquina como cliente do serviço DNS. Para isso vamos configurar as interfaces de rede e adicionar os IPs de DNS e o compo "search".

Vamos entrar no arquivo de configuração:

```
$ sudo nano /etc/netplan/00-installer-config.yaml
```

**RESULTADO**

```
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens160:
      addresses: [10.9.14.103/22]
      gateway4: 10.9.14.1
      dhcp4: false
      nameservers:
        addresses:
          - 10.9.14.126                                 # IP ns1 (pedro)
          - 10.9.14.109                                 # IP ns2 (filipe)
        search: [grupo4.turma914.ifalara.local]
    ens192:
      addresses: [192.168.14.25/29]
  version: 2
```

![installer_confing](/gw/installer_confing.PNG)

Em seguida vamos aplicar as alterações.

Para isso usamos:

```
$ sudo netplan apply
```
---

### 2 - Agora vamos testar se está funcionando normalmente.

```
$ dig @10.9.14.126 ns2.grupo4.turma914.ifalara.local
```

Na linha "ANSWER SECTION:" tem que aparecer que foi resolvido, com o dominio e o IP.

Vamos usar o seguinte comando para verificar se a interface 160 está funcionando:

```
$ systemd-resolve --status ens160
```

Para finalizar vamos ver se o nosso serviço DNS revolve o DNS do google:

$ ping google.com

![dig-system_resolve-ping](/gw/dig-system_resolve-ping.PNG)

---


### 3 - Vamos habilitar o firewall

Para ativar o fireall e permitir o acesso ao shh:

```
$ sudo ufw enable
```
```
$ sudo ufw allow ssh
```

Vamos habilitar o encaminhamento das interfaces **WAN** e **LAN**

Para isso vamos retirar a marcação de comentário da linha: ``# net/ipv4/ip_forward=1``

```
$ sudo nano /etc/ufw/sysctl.conf
```
```
net/ipv4/ip_forwarding=1
```

Como o arquivo de configuração /etc/rc.local não existe vamos ter que cria-lo:

```
$ sudo nano /etc/rc.local
```

```
#!/bin/bash

# /etc/rc.local

# Default policy to drop all incoming packets.
# Politica padrão para bloquear (drop) todos os pacotes de entrada
iptables -P INPUT DROP
iptables -P FORWARD DROP

# Accept incoming packets from localhost and the LAN interface.
# Aceita pacotes de entrada a partir das interfaces localhost e the LAN.
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -i ens192 -j ACCEPT

# Accept incoming packets from the WAN if the router initiated the connection.
# Aceita pacotes de entrada a partir da WAN se o roteador iniciou a conexao
iptables -A INPUT -i ens160 -m conntrack \
--ctstate ESTABLISHED,RELATED -j ACCEPT

# Forward LAN packets to the WAN.
# Encaminha os pacotes da LAN para a WAN
iptables -A FORWARD -i ens192 -o ens160 -j ACCEPT

# Forward WAN packets to the LAN if the LAN initiated the connection.
# Encaminha os pacotes WAN para a LAN se a LAN inicar a conexao.
iptables -A FORWARD -i ens160 -o ens192 -m conntrack \
--ctstate ESTABLISHED,RELATED -j ACCEPT

# NAT traffic going out the WAN interface.
# Trafego NAT sai pela interface WAN
iptables -t nat -A POSTROUTING -o ens160 -j MASQUERADE

# rc.local needs to exit with 0
# rc.local precisa sair com 0
exit 0
```

![rc_local](/gw/rc_local.PNG)

Vamos tornar o arquivo rc.local em executável e inicializável no boot

```
$ sudo chmod 755 /etc/rc.local
```

Vamos verificar se o firewall está funcionando

```
$ sudo ufw status
```

ou

```
$ systemctl status ufw.service
```

Vamos reiniciar a máquina

```
$ sudo reboot
```

### 5 - Agora vamos fazer o encaminhamento de portas para o acesso externo:

##### O serviço SAMBA utiliza as Portas: ``445`` e ``139`` na interface externa ``ens160``, nosso samba tem o IP: ``10.9.14.108``

Então vamos adicionar os seguintes blocos de codigos em rc.local

```
#Recebe pacotes na porta 445 da interface externa do gw e encaminha para o servidor interno na porta 445
iptables -A PREROUTING -t nat -i ens160 -p tcp --dport 445 -j DNAT --to 10.9.14.108:445
iptables -A FORWARD -p tcp -d 10.9.14.108 --dport 445 -j ACCEPT

#Recebe pacotes na porta 139 da interface externa do gw e encaminha para o servidor interno na porta 139
iptables -A PREROUTING -t nat -i ens160 -p tcp --dport 139 -j DNAT --to 10.9.14.108:139
iptables -A FORWARD -p tcp -d 10.9.14.108 --dport 445 -j ACCEPT
```

###### O serviço DNS utiliza a Porta: ``53`` na interface externa ``ens160``, nosso ns1 tem o IP: ``10.9.14.126``

```
#Recebe pacotes na porta 53 da interface externa do gw e encaminha para o servidor DNS Master interno na porta 53
iptables -A PREROUTING -t nat -i ens160 -p tcp --dport 53 -j DNAT --to 10.9.14.126:53
iptables -A FORWARD -p udp -d 10.9.14.126 --dport 53 -j ACCEPT
```

###### O serviço Web utiliza a Porta: ``80`` na interface externa ``ens160``, nosso www tem o IP: ``10.9.14.217``

```
#Recebe pacotes na porta 80 da interface externa do gw e encaminha para o servidor www interno na porta 80
iptables -A PREROUTING -t nat -i ens160 -p tcp --dport 80 -j DNAT --to 10.9.14.217:80
iptables -A FORWARD -p tcp -d 10.9.14.217 --dport 80 -j ACCEPT
```

Então vamos adicionar em rc.local

![rc_local](/gw/rc_local.PNG)

![rc_local_portas](/gw/rc_local_portas.PNG)

```
$ sudo nano /etc/rc.local
```

### 6 - Agora vamos comentar todas as máquinas: ``ns1``, ``ns2``, ``smb``, ``www`` e ``bd`` a nosssa maquina ``gw`` que é o nosso gateway

```
$ sudo nano /etc/netplan/00-installer-config.yaml
```

##### Na interface ens160 vamos comentar

```
#gateway4: 10.9.14.1
```

##### Na interface ens192 vamos adicionar

```
gateway4: 192.168.14.125
```

![installer_confing_ns1](/gw/installer_confing_ns1.PNG)

![installer_confing_ns2](/gw/installer_confing_ns2.PNG)

![installer_confing_smb](/gw/installer_confing_smb.PNG)

![installer_confing_www](/gw/installer_confing_www.PNG)

![installer_confing_bd](/gw/installer_confing_bd.PNG)

### Considerações Finais

Após termos editado tudo todos os serviços vão precisar passar pela máquina "gw", ou sejá, as maquinas não vão ser acessadas normalmente via ssh, pois terá que passar pelo gateway

Para acessar as máquinas ``ns1``, ``ns2``, ``smb``, ``www`` e ``bd`` vamos ter que conectar primeiro com a nossa maquina ``gw``

Assim também os nossos DNS, para acessar o samba ou o nosso site web pelo nome tem que ser colocado o nome do ``gw``

Para que o nosso serviço samba funcione com o nome correto devemos ir na máquina ``ns1`` no arquivo de zona direta e adicionar um comentario no nome da maquina ``smb`` e fazer referencia a ele pelo ``gw`` com CNAME 

#### Na maquina **ns1**

Vamso editar o arquivo db.grupo4.turma914.ifalara.local comentando com ``;`` a linha ``smb.grupo4.turma914.ifalara.local.`` e adicoinando o CNAME

Vamso editar o arquivo

```
$ sudo /etc/bind/zones/db.grupo4.turma914.ifalara.local
```

**RESULTADO**
```
;smb.grupo4.turma914.ifalara.local.       IN    A       10.9.14.108
smb CNAME gw
```

![zonaDireta_canme](/gw/zonaDireta_canme.PNG)

Para checar usamos:

```
$ sudo named-checkzone grupo4.turma914.ifalara.local db.grupo4.turma914.ifalara.local
```

E então damos um restar no bind9

```
$ sudo systemctl restart bind9
```

![smb_gw_nome_teste](/gw/smb_gw_nome_teste.PNG)




# Considerações Finais

Aqui conseguimos fazer a interação entre seis maquinas virtuais utilizando ferramentas 
