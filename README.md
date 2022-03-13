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
Texto deintrodução

# Definições Iniciais

### CONFIGURAÇÕES DAS INTEFACES DE REDE

| Rede | IP Externo | Mascara | IP Interno | Mascara |
| --- | --- | --- | --- | --- |
| IP da Subrede: | 10.9.14.0 | /24 | 192.168.14.24 | /29 |
| IP de Broadcast: | 10.9.14.255/24 | /24 | 192.168.14.31	| /29 |

| Nome da VMs | WAN | IP | LAN | IP |
| --- | --- | --- | --- | --- |
| IP do GW: | ens160 | 10.9.14.108 | ens192 |	192.168.14.25 |
| IP do SAMBA: | ens160 | 10.9.14.103 | ens192 |	192.168.14.26 |
| IP do NS1: | ens160 | 10.9.14.126 | ens192 |	192.168.14.27 |
| IP do NS2: | ens160 | 10.9.14.109 | ens192 |	192.168.14.28 |
| IP do WEB | ens160 | 10.9.14.217 | ens192 |	192.168.14.29 |
| IP do BD | ens160 | 10.9.14.218 | ens192 |	192.168.14.30 |

### DEFINIÇÂO DE NOMES E DOMÍNIO (grupo4.turma914.ifalara.local):				

| VM | Nome | Domínio (zona): | grupo4.turma914.ifalara.local |
| --- | --- |--- | --- |
| Aluno08 | Elias | FQDN do GW: | gw.grupo4.turma914.ifalara.local |
| Aluno03 | Bruno |	FQDN do SAMBA: | smb.grupo4.turma914.ifalara.local |
| Aluno26	| Pedro | FQDN do NS1: | ns1.grupo4.turma914.ifalara.local |
| Aluno09 | Filipe | FQDN do NS2: | ns2.grupo4.turma914.ifalara.local |	
| Grupo4vm1	| --- | FQDN do WEB	| www.grupo4.turma914.ifalara.local |	
| Grupo4vm2 | --- |	FQDN do BD | bd.grupo4.turma914.ifalara.local |

# Implementação dos Serivços de Rede (Cada serviço uma sessão)

## Gateway

## Samba

## DNS 

### Configuração do Bind9 (DNS Server)

#### Master

  Nesta maquina ns1 vamos configurar o serviço de resolução de nome (DNS) com o Bind9, utilizando à como DNS Master. 

#### 0 - Antes de tudo vamos mudar o nome da maquina para o nome escolhido que está na tabela:

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
  $ sudo mkdir /var/www/grupo4.turma914.ifalara.local
  ```
  
  Agora atribuimos a propriedade do diretório com a variável de ambiente $USER:
  
  ```
  $ sudo chown -R $USER:$USER /var/www/grupo4.turma914.ifalara.local
  ```
  
  Agora vamos configurar as permições dos webs hosts, com:
  
  ```
  $ sudo chmod -R 755 /var/www/grupo4.turma914.ifalara.local
  ```
  
  Agora vomos criar e editar a pagina index.html:
  
  ```
  $ sudo nano /var/www/gupo4.turma914.ifalara.local/index.html
  ```
  
  Para que o Apache apresente a nossa página vamos criar um arquivo de configuração novo, com:
  
  ```
  $ sudo nano /etc/apache2/sites-available/grupo4.turma914.ifalara.local.conf
  ```
  
  Neste arquivo vamos colocar o seguinte bloco de cofigo:
  
  ```
  <VirtualHost *:80>
      ServerAdmin webmaster@localhost
      ServerName grupo4.turma914.ifalara.local
      ServerAlias www.grupo4.turma914.ifalara.local
      DocumentRoot /var/www/grupo4.turma914.ifalara.local
      ErrorLog ${APACHE_LOG_DIR}/error.log
      CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```
  
  ![domain_conf](/www/domain_conf.PNG)

  Em segida vamos habilitar o arquivo com a ferramenta a2ensite:
  
  ```
  $ sudo a2ensite grupo4.turma914.ifalara.local.conf
  ```
  
  Vamos desabilitar o site padrão definido em 000-default.conf:
  
  ```
  $ sudo a2dissite 000-default.conf
  ```
  
  Em seguida, vamos testar à procura de erros de configuração:
  
  ```
  $ sudo apache2ctl configtest
  ```
  
  Vamos reiniciar o Apache com:
  
  ```
  $ sudo systemctl restart apache2
  ```
  
  Agora é só acessar novamente no navegador com o IP da maquina. 
  
  ![a2ensite-configtest-restart](/www/a2ensite-configtest-restart.PNG)
  
  ![site](/www/site.PNG)

--

## Banco de Dados

  Nesta maquina "bd" vamos configurar o servidor de banco de dados usando a ferramenta MySQL e configurar a maquina como cliente do serviço DNS. 

### 0 - Primeiro vamos mudar o nome da maquina para o nome escolhhido que está na tabela:

  > bd.grupo4.turma914.ifalara.local

  Com o comando:
  
  ```
  $ sudo hostnamectl set-hostname bd.grupo4.turma914.ifalara.local
  ```
  
  ![hostname](/bd/hostname.PNG)

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
  
  ![installer_confing](/bd/installer_confing.PNG)

  Em seguida vamos aplicar as alterações.

  Para isso usamos:

  ```
  $ sudo netplan apply
  ```
  
### 2- Agora vamos testar se está funcionando normalmente.

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
  
  ![dig-system_resolve-ping](/bd/dig-system_resolve-ping.PNG)

--


# Considerações Finais
