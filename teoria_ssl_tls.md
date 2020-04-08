# Explicació claus

## Criptografia

Podem classificar la segurat amb el tipus de criptografia a utilitzar:

* `Criptografia de clau simètrica`

* `Criptografia de clau asimètrica`

* `Criptografia híbrida`

### Claus simètriques

S'anomena criptografia de **clau simètrica** quan s'utilitza la **matèixa** clau per **encriptar** com per **desencriptar**. La clau o secret ha de ser compartida per l'emissor i pel receptor (encriptar/desencriptar respectivament).

Les claus d'aquest tipus són fàcils de gestionar en el sentit de que permeten una seguretat molt robusta amb claus *petites*(pocs bits comparades amb les *asimètriques*) i un cost algorísmic menor.

El problema del xifrat *simètric* és que aquesta clau/secret sigui compartit entre els interlocutors, per tant segueix existint el problema de **com compartir aquest secret de forma segura**.

Proporciona un bon rendiment per xifrar **fluxos continus de dades**, per xifrar **comunicacions TCP**.

Són algorismes de clau simètrica **DES**, **3DES**, **AES** i **IDEA** entre altres.

### Claus asimètriques

S'anomena criptografia de **clau asimètrica** al fet d'utilitzar dues claus diferents una anomenada *clau privada* i una altre *clau pública*. Les dues parelles s'utilitzen com una parella per aconseguir les finalitats de **xifrar i signar**.

# Exemples d'ordres

### Claus privades RSA

#### Claus privades RSA

```bash
#
openssl genrsa -des3 -out ca.key 2048
#
openssl genrsa -nodes -out server.key 2048
```

#### Passfrase des3

```bash
#
openssl rsa -des3 -in server.key -out passfrase.server.key
#
openssl rsa -in passfrase.server.key -out deleted-passfrase.server.key
#
openssl rsa -des3 -in passfrase.server.key -out new-passfrase.server.key
```

#### Llistar

```bash
#
openssl rsa -noout -text -in serverkey-pem
cat serverkey.pem
```

#### Conversió PEM / DER

```bash
# Ho podem fer sense especificar el format actual
openssl rsa -in key.pem -outform DER -out key.der
# Podem fer la conversió especificant el nou format
openssl rsa -inform DER -in key.der -outform PEM -out key.pem
# O sense especificar-lo
openssl rsa -inform DER -in key.der -out key.pem
```

#### Extreure la clau pública de la privada

```bash
#
openssl rsa -in key.pem -pubout -out pubkey.pem
```

### Certificats digitals X509

#### Certificat autosignat (genera cert i key)

```bash
# 
openssl req -new -x509 -nodes -out servercert.pem -keyout serverkey.pem
#
openssl req -new -x509 -out servercert.pem -keyout passfrasse.serverkey.pem
#
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem
#
openssl req -x509 -nodes -days 365 -sha256 \
    -subj '/C=US/ST=Oregon/L=Portland/CN=www.madboa.com' \
    -newkey rsa:2048 -keyout mycert.pem -out mycert.pem
```

#### Certificat autosignat usant una clau privada existent (cakey.pem)

```bash
#
openssl req -new -x509 -days 365 -key cakey.pem -out cacert.pem
```

#### Petició de certificat (request)

```bash
#
openssl req -new -key serverkey.pem -out serverreq-pem 
```

#### CA signar un request/ Generar X509

```bash
#
openssl x509 -CA cacert.pem -CAkey cakey.pem -req -in serverreq.pem -out servercert.pem
#
openssl x509 -CA cacert.pem -CAkey cakey.pem -req -in serverreq.pem |
    -days 365 -extfile ca.conf -CAcreateserial -out servercert.pem
```

#### Definir extensions en un fitxer

```bash
#
cat ca.conf
basicConstraints = critical,CA:FALSE
extendedKeyUsage = serverAuth,emailProtection
```
