# Explicació claus

## Criptografia

Podem classificar la segurat amb el tipus de criptografia a utilitzar:

* `Criptografia de clau simètrica`

* `Criptografia de clau asimètrica`

* `Criptografia híbrida`

### Criptografia Claus simètriques

S'anomena criptografia de **clau simètrica** quan s'utilitza la **matèixa** clau per **encriptar** com per **desencriptar**. La clau o secret ha de ser compartida per l'emissor i pel receptor (encriptar/desencriptar respectivament).

Les claus d'aquest tipus són fàcils de gestionar en el sentit de que permeten una seguretat molt robusta amb claus *petites*(pocs bits comparades amb les *asimètriques*) i un cost algorísmic menor.

El problema del xifrat *simètric* és que aquesta clau/secret sigui compartit entre els interlocutors, per tant segueix existint el problema de **com compartir aquest secret de forma segura**.

Proporciona un bon rendiment per xifrar **fluxos continus de dades**, per xifrar **comunicacions TCP**.

Són algorismes de clau simètrica **DES**, **3DES**, **AES** i **IDEA** entre altres.

### Criptografia Claus asimètriques

S'anomena criptografia de **clau asimètrica** al fet d'utilitzar dues claus diferents una anomenada *clau privada* i una altre *clau pública*. Les dues parelles s'utilitzen com una parella per aconseguir les finalitats de **xifrar i signar**. No s'ha d'interpretar com que una només xifra i l'altra només desxifra, sinó com que allò que s'ha 'fet' amb una es pot 'desfer' amb l'altre.

Cada interlocutor ha de disposar de la seva parella de claus privada/pública. La clau **privada** s'ha de mantenir privada, fora de l'abast de qualsevol.

Per altre bada, la clau **pública** és la que compartim amb tothom. Aixi, per exemple, per xifrar es **xifra amb la clau pública** del destinatari i el destinatari la **desxifra amb la seva privada**.

Alhora de signar digitalment es **signa amb la clau privada** de l'emissor i el receptor verifica la firma amb la **clau pública**.

La principal diferència que té amb les claus simètriques és que no ens hem de preocupar en com enviem les claus, ja que sempre tenim la privada guardada i compartim la pública.

Aquest model és bo per xifrar continguts concrets com per exemple emails però **massa costos** de càlcul per poder xifrar comunicacions de dades continues com una sessió TCP.

Són algorismes de clau asimètrica **RSA**, **DSA** i **ElGamal** entre altres.

### Criptografia híbrida

Tal i com s'ha exposat la criptografia **simètrica és més ràpida i menys costosa** computacionalment que la criptografia **asimètrica**, però té l'inconvenient que cal que els dos interlocutors **coneguin la clau simètrica**. Un mecanisme molt usat per a comunicacions segures intenta aprofitar els avantatges dels dos models anteriors.

Així, per exemple, tant **SSH** com **SSL** combinen els dos tipus en un model anomenat **híbrida**. El model *asimètric* (basat en els certificats digitals de claus privada/pública) permet que els dos interlocutors puguin iniciar una comunicació segura sense necessitat de cap secret compartit. Un cop establerta la comunicació segura els dos interlocutors poden intercanviar-se un 'secret compartit' e iniciar una comunicació xifrada *simètrica* que proporciona un rendiment millor pel xifrat d'un diàleg TCP. 

## Tipus de claus

També podem fer una altre classificació basada en claus:

* `Privada`

* `Pública`

* `Certificats`

### Clau privada

Una clau privada és una clau usualment del tipus **RSA** o **DSA** d'un número de bits variable (actualment s'utilitzen valors de 1024, 2048 o 4094). Aquesta clau **s'ha** de mantenir fora de l'abast de tothom (excepte del seu propietari). Es pot afegir un extra de seguretat xifrant el fitxer de la clau amb un password (xifrat simètric perque el mateix password xifra/desxifra).

### Clau pública

La clau pública és la que podem compartir amb tothom, com tal sola no és útil.

Les claus públiques van aparellades a les claus privades i formen una parella *indissoluble*.

Cal tenir la parella privada/pública per poder implementar seguretat amb certificats digitals.

### Certificats

Les claus públiques es transformen en **certificats digitals** que són els que realment s'utilitzen. En general quan parlem de *clau pública* ens referim al *certificat digital*.

Un certificat digital no és més que una clau pública **avalada** per una entitat. Aixi, un certificat digital és una clau pública signada per una entitat que avala com a vàlid el certificat.

Aquest certificat és el que s'ha de fer arribar als altres interlocutors.

El format actual és **X509**.

Ens podem preguntar per què no s'utilitzen directament les claus públiques en lloc dels certificats, i la resposta és molt simple, perquè la clau pública d'en *pere* no sabem si realment és d'en *pere* o d'algú que s'hi fa passar. En canvi, un certificat d'en *pere* és una clau pública on una autoritat ha avalat que realment és en *pere*.

El certificat es firma per una **CA** o per un mateix (auto-certificats). Per firmar es fa un hash i se li aplica la clau privada de la **CA** (generalment algorisme **RSA**).

## Formats dels fitxers

Podem trobar diferents formats en els fitxers:

* `PEM`

* `DER`

* `xifrat`

* `altres`

### Format PEM

Aquest és el format més usual i el format per defecte d'eines com **openssl**.

Conté les claus (privada o pública) en format **ascii**. De fet, el contingut binari de les claus (format **DER**) es cofidica en *base64* per generar un text *ascii* imprimible i se li afegeixen capçaleres i peus identificadors del tipus de clau.

### Format DER

Contenen les claus en format **binari**.



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
