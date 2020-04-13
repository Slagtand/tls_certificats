# Servidor ldap segur

## Generar certificats

### Generar claus privades del servidor

```bash
$ openssl genrsa -out cakey.pem 2048
$ openssl genrsa -out serverkey.pem 2048
```

### Generar certificat propi de la CA

```bash
[marc@localhost ldapserver19:tls]$ openssl req -new -x509 -nodes -sha1 -days 365 -key cakey.pem -out cacert.pem
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter &#39;.&#39;, the field will be left blank.
-----
Country Name (2 letter code) [XX]:ca
State or Province Name (full name) []:Barcelona
Locality Name (eg, city) [Default City]:Barcelona
Organization Name (eg, company) [Default Company Ltd]:Veritat Absoluta
Organizational Unit Name (eg, section) []:Certificats
Common Name (eg, your name or your server&#39;s hostname) []:VeritatAbsoluta
Email Address []:admin@edt.org
```

### Generar un certificat request per enviar a la certificadora CA

```bash
[marc@localhost ldapserver19:tls]$ openssl req -new -key serverkey.pem -out servercert.pem
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter &#39;.&#39;, the field will be left blank.
-----
Country Name (2 letter code) [XX]:ca
State or Province Name (full name) []:Barcelona
Locality Name (eg, city) [Default City]:Barcelona
Organization Name (eg, company) [Default Company Ltd]:EDT
Organizational Unit Name (eg, section) []:departament informatica
Common Name (eg, your name or your server&#39;s hostname) []:ldap.edt.org
Email Address []:admin@edt.org

Please enter the following &#39;extra&#39; attributes
to be sent with your certificate request
A challenge password []:jupiter
An optional company name []:edt
```

### Creació CA

Nosaltres mateixos farem de CA

```bash
[marc@localhost ldapserver19:tls]$ cat ca.conf 
basicConstraints = critical,CA:FALSE
extendedKeyUsage = serverAuth,emailProtection
```

### Firma certificat com a CA

```bash
[marc@localhost ldapserver19:tls]$ openssl x509 -CA cacert.pem -CAkey cakey.pem -req -in servercsr.pem -days 365 -sha1 -extfile ca.conf -CAcreateserial -out servercert.pem
Signature ok
subject=C = ca, ST = Barcelona, L = Barcelona, O = EDT, OU = departament informatica, CN = ldap.edt.org, emailAddress = admin@edt.org
Getting CA Private Key
```

## Configuracions per TLS

Per a generar una connexió segura encara que ens connectem a un port insegur hem de configurar alguns fitxers:

### Configuració del server

Al server hem de tocar els següents fitxers:

- fitxer `slapd.conf`: hem d'afegir les següents línies
  
  ```bash
  TLSCACertificateFile    /etc/openldap/certs/cacert.pem
  TLSCertificateFile      /etc/openldap/certs/servercert.pem
  TLSCertificateKeyFile   /etc/openldap/certs/serverkey.pem
  TLSVerifyClient         never
  TLSCipherSuite          HIGH:MEDIUM:LOW:+SSLv2
  ```

- fitxer `ldap.conf`: hem d'afegir (i editar on sigui necessari) les següents línies
  
  ```bash
  TLS_CACERT /etc/openldap/certs/cacert.pem
  SASL_NOCANON    on
  URI ldap://ldap.edt.org
  BASE dc=edt,dc=org
  ```

- Per finalitzar, editem l'arrencada del servei al fitxer `startup.sh`:
  
  ```bash
  /sbin/slapd -d0 -h "ldap:/// ldaps:/// ldapi:///"
  ```

### Configuració del client

Per la part del client, hem de seguir les següents indicacions.

- Si tenim el servidor en un docker, hem d'afegir l'entrada corresponent a `/etc/hosts`.

- Copiar el certificat de la CA que hem creat abans a `/etc/openldap/cacert.pem`.

- Editem el fitxer `/etc/openldap/ldap.conf`: indiquem on es troba el certificat anterior i toquem les següents línies
  
  ```bash
  TLS_CACERT /etc/openldap/cacert.pem
  TLS_REQCERT allow
  
  URI ldap://ldap.edt.org
  BASE dc=edt,dc=org
  
  SASL_NOCANON on
  ```

## Comprovacions

Comprovem amb les ordres `ldapsearch` i `openssl` si ho hem fet tot correctament:

- Amb `ldapsearch` hi afegirem l'opció `-ZZ` que, bàsicament, executa TLS i **requereix** que l'operació sigui satisfactoria per executar l'ordre.

```bash
[root@localhost ldapserver19:tls]# ldapsearch -x -LLL -ZZ -h ldap.edt.org -b &#39;dc=edt,dc=org&#39; dn
dn: dc=edt,dc=org

dn: ou=usuaris,dc=edt,dc=org

dn: cn=Pau Pou,ou=usuaris,dc=edt,dc=org

dn: cn=Pere Pou,ou=usuaris,dc=edt,dc=org

dn: cn=Anna Pou,ou=usuaris,dc=edt,dc=org

...

# Amb -H indiquem una URI
[root@localhost ldapserver19:tls]# ldapsearch -x -LLL -H ldaps://ldap.edt.org dn
dn: dc=edt,dc=org

dn: ou=usuaris,dc=edt,dc=org

dn: cn=Pau Pou,ou=usuaris,dc=edt,dc=org

dn: cn=Pere Pou,ou=usuaris,dc=edt,dc=org

dn: cn=Anna Pou,ou=usuaris,dc=edt,dc=org

dn: cn=Marta Mas,ou=usuaris,dc=edt,dc=org

dn: cn=Jordi Mas,ou=usuaris,dc=edt,dc=org

...
```

- Amb `openssl` comprovem que connecta amb el servidor i els certificats d'aquest

```bash
[root@localhost ldapserver19:tls]# openssl s_client -connect ldap.edt.org:636
CONNECTED(00000003)
depth=1 C = ca, ST = Barcelona, L = Barcelona, O = Veritat Absoluta, OU = Certificats, CN = VeritatAbsoluta, emailAddress = admin@edt.org
verify error:num=19:self signed certificate in certificate chain
verify return:1
depth=1 C = ca, ST = Barcelona, L = Barcelona, O = Veritat Absoluta, OU = Certificats, CN = VeritatAbsoluta, emailAddress = admin@edt.org
verify return:1
depth=0 C = ca, ST = Barcelona, L = Barcelona, O = EDT, OU = departament informatica, CN = ldap.edt.org, emailAddress = admin@edt.org
verify return:1
---
```

# OpenVPN amb certificats propis

## Generaració de claus al servidor

### Creació CA

Primer crearem el nostre CA amb la seva clau i certificat

```bash
# Ens demanarà una contrasenya per poguer-la certificar després
openssl genrsa -des3 -out cakey.pem 2048

# Utilitzarem la contrasenya que hem ficat a l'ordre anterior
openssl req -new -x509 -nodes -sha1 -days 365 -key cakey.pem -out cacert.pem
Enter pass phrase for cakey.pem:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:ca
State or Province Name (full name) []:Barcelona
Locality Name (eg, city) [Default City]:Barcelona
Organization Name (eg, company) [Default Company Ltd]:Veritat Absoluta
Organizational Unit Name (eg, section) []:Dep certificats
Common Name (eg, your name or your server's hostname) []:VeritatAbsoluta
Email Address []:admin@edt.org
```

### Generació clau del servidor i request per la CA

Generarem la clau i el request del servidor per la CA que hem creat

```bash
openssl dhparam -out dh2048.pem 2048

# Clau del servidor
openssl genrsa -out serverkey.pem 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
.........................................+++++
........................................................................................................................+++++
e is 65537 (0x010001)

# Request per la CA
openssl req -new -key serverkey.pem -out serverreq.pem
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:ca
State or Province Name (full name) []:Barcelona
Locality Name (eg, city) [Default City]:Barcelona
Organization Name (eg, company) [Default Company Ltd]:Server vpn
Organizational Unit Name (eg, section) []:Server vpn
Common Name (eg, your name or your server's hostname) []:ServerVpn
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:jupiter
An optional company name []:edt
```

### Creació i certificació de CA externa

Al no tindre una CA externa en simularem una i ens "auto certificarem". 

Utilitzarem un fitxer anomenat `ca.externa.conf` que contindrà el següent:

```bash
basicConstraints       = CA:FALSE
nsCertType             = server
nsComment              = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid,issuer:always
extendedKeyUsage       = serverAuth
keyUsage               = digitalSignature, keyEncipherment
```

Generem el certificat

```bash
openssl x509 -CAkey cakey.pem -CA cacert.pem -req -in serverreq.pem -days 365 -CAcreateserial -extfile ca.externa.conf -out servercert.pem
Signature ok
subject=C = ca, ST = Barcelona, L = Barcelona, O = Server vpn, OU = Server vpn, CN = ServerVpn
Getting CA Private Key
```

## Generació de claus al client

Aquest procés s'ha de fer per cada client, ja que cada client hauria de tindre una clau pròpia per conectar al servidor.

### Generació de clau client i request per la CA

```bash
[marc@localhost client]$ openssl genrsa -out keyclie1.pem 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
...............+++++
..................................+++++
e is 65537 (0x010001)
[marc@localhost client]$ openssl req -new -key keyclie1.pem -out reqclie1.pem
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:ca
State or Province Name (full name) []:Barcelona
Locality Name (eg, city) [Default City]:Barcelona
Organization Name (eg, company) [Default Company Ltd]:client 1 vpn
Organizational Unit Name (eg, section) []:client1
Common Name (eg, your name or your server's hostname) []:client1
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:jupiter
An optional company name []:edt
```

### Certificació de CA externa

Com en el cas del servidor, hauria de ser una CA externa, però la simularem.

Utilitzarem un fitxer anomenat `client.externa.conf` que contindrà:

```bash
basicConstraints        = CA:FALSE
subjectKeyIdentifier    = hash
authorityKeyIdentifier  = keyid,issuer:always
```

Generem el certificat pel client

```bash
[marc@localhost client]$ openssl x509 -CAkey cakey.pem -CA cacert.pem -req -in reqclie1.pem -days 365 -CAcreateserial -extfile client.externa.conf -out certclie1.pem
Signature ok
subject=C = ca, ST = Barcelona, L = Barcelona, O = client 1 vpn, OU = client1, CN = client1
Getting CA Private Key
```

## Configuració vpn i arrencada servidor

Un cop creats totes les claus i certificats corresponents, passem a la configuració del servidor.

* Primer modificarem el servei per utilitzar el fitxer de configuració que crearem més endavant

```bash
# Creem una còpia abans de modificar res
[fedora@ip-172-31-81-94 seguretat-aws]$ sudo cp /lib/systemd/system/openvpn-server@.service /etc/systemd/system/.

# Deixem el contingut així
[fedora@ip-172-31-81-94 seguretat-aws]$ cat /etc/systemd/system/openvpn-server\@.service 
[Unit]
Description=OpenVPN service for %I hisx
After=syslog.target network-online.target

[Service]
Type=forking
PrivateTmp=true
ExecStartPre=/usr/bin/echo serveri %i %I
PIDFile=/var/run/openvpn-server/%i.pid
ExecStart=/usr/sbin/openvpn --daemon --writepid /var/run/openvpn-server/%i.pid --cd /etc/openvpn/ --config %i.conf

[Install]
WantedBy=multi-user.target

```

* Agafant el fitxer de configuració de mostra `/usr/share/doc/openvpn/sample/sample-config-files/server.conf` el modifiquem per a que contingui la següent conf i el copiem a `/etc/openvpn/confserver.conf`

* Al ser una màquina aws hem de recordar **obrir** el port corresponent a l'apartat *security groups*

```bash
[fedora@ip-172-31-81-94 seguretat-aws]$ cat /etc/openvpn/confserver.conf 

port 1194
proto udp
# interfície de la vpn
dev tun

# Les claus i certificats del servidor
ca /etc/openvpn/keys/cacert.pem
cert /etc/openvpn/keys/servercert.pem
key /etc/openvpn/keys/serverkey.pem 
dh /etc/openvpn/keys/dh2048.pem

# Xarxa de la vpn
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
# Els clients es poden veure entre ells
client-to-client
# No permetem diferents connexions amb la mateixa clau
;duplicate-cn

keepalive 10 120
cipher AES-256-CBC
comp-lzo

persist-key
persist-tun

# dades dels logs
status openvpn-status.log
verb 3
explicit-exit-notify 1
```

* Per la part del servidor l'estructura queda així

```bash
[fedora@ip-172-31-81-94 seguretat-aws]$ sudo tree /etc/openvpn
/etc/openvpn
├── client
├── confserver.conf
├── ipp.txt
├── keys
│   ├── cacert.pem
│   ├── dh2048.pem
│   ├── servercert.pem
│   └── serverkey.pem
├── openvpn-status.log
└── server

3 directories, 7 files
```

Per finalitzar, arrancarem el servidor.

* Hem de tindre en compte que hem modificat el servei, pel que hem de carregar-lo de nou.

* Al encendre'l hem de posar el nom del fitxer de **configuració que hem creat**.

```bash
# Recarreguem els dominis
[fedora@ip-172-31-81-94 seguretat-aws]$ sudo systemctl daemon-reload 

# Arrenquem el servidor
[fedora@ip-172-31-81-94 seguretat-aws]$ sudo systemctl start openvpn-server@confserver.service
```

Comprovem que l'interfície s'ha creat correctament

```bash
[fedora@ip-172-31-81-94 seguretat-aws]$ ip a s tun0
5: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 100
    link/none 
    inet 10.8.0.1 peer 10.8.0.2/32 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::874f:5ba0:2eae:8bae/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever

```

## Configuració vpn i arrencada client

Al client només hem de tocar la configuració que ficarem a `/etc/openvpn/client/` i arrencar el servei.

* Això s'ha de fer per cada client que afegim

* Hem de tindre en compte que podem indicar la **ip** del servidor o un nom de **host**. Si fem servir el nom de host ens hem d'enrecordar d'afegir la resolució a `/etc/hosts`

```bash
[marc@localhost client]$ cat confclient.conf 
client
dev tun
proto udp

# ip o host amb el port del servidor
remote aws 1194
resolv-retry infinite
nobind
persist-key
persist-tun

# Les claus i certificats del client
ca /etc/openvpn/keys/cacert.pem
cert  /etc/openvpn/keys/certclie1.pem
key  /etc/openvpn/keys/keyclie1.pem

remote-cert-tls server
cipher AES-256-CBC
comp-lzo
verb 3
```

* Si hem fet servir nom de host editem `/etc/hosts`

```bash
[marc@localhost client]$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
#172.17.0.2  ldap.edt.org
3.92.239.6   aws
```

* Per la part del client l'estructura queda aixi

```bash
[marc@localhost client]$ sudo tree /etc/openvpn/
/etc/openvpn/
├── client
│   └── confclient.conf
├── keys
│   ├── cacert.pem
│   ├── certclie1.pem
│   └── keyclie1.pem
└── server

3 directories, 4 files
```

Arrenquem el client

```bash
[marc@localhost client]$ sudo systemctl start openvpn-client@confclient.service
```

Comprovem que s'ha creat l'interfície

```bash
[marc@localhost client]$ sudo ip a s tun0
6: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 100
    link/none 
    inet 10.8.0.6 peer 10.8.0.5/32 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::9089:2213:5588:3b99/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever

```

## Resultat final

Per acabar, comprovem que es poden comunicar entre ells a través del túnel que hem creat fent un ping a la ip de cada extrem.

* Del client al servidor

```bash
[marc@localhost client]$ ping 10.8.0.1
PING 10.8.0.1 (10.8.0.1) 56(84) bytes of data.
64 bytes from 10.8.0.1: icmp_seq=1 ttl=64 time=123 ms
64 bytes from 10.8.0.1: icmp_seq=2 ttl=64 time=325 ms
64 bytes from 10.8.0.1: icmp_seq=3 ttl=64 time=272 ms
64 bytes from 10.8.0.1: icmp_seq=4 ttl=64 time=162 ms
64 bytes from 10.8.0.1: icmp_seq=5 ttl=64 time=389 ms
...
```

* Del servidor al client

```bash
[fedora@ip-172-31-81-94 seguretat-aws]$ ping 10.8.0.6
PING 10.8.0.6 (10.8.0.6) 56(84) bytes of data.
64 bytes from 10.8.0.6: icmp_seq=1 ttl=64 time=207 ms
64 bytes from 10.8.0.6: icmp_seq=2 ttl=64 time=122 ms
64 bytes from 10.8.0.6: icmp_seq=3 ttl=64 time=131 ms
64 bytes from 10.8.0.6: icmp_seq=4 ttl=64 time=127 ms
...
```
