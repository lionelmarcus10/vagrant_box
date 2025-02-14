## Install openldap and config
```bash
sudo apt-get install -y slapd
```
### verify version 
```bash
slapd -VV
```
### get process 
```bash
ss -ltnp
# alternative
netstat -laptn | grep slapd
```
### mofify ldap conf

#### Part 1
```bash
nano  /etc/ldap/ldap.conf

# put
BASE    dc=Efrei,dc=fr
URI     ldap://Efrei.fr

sudo echo '127.0.0.1 localhost Efrei.fr' | sudo tee -a /etc/hosts > /dev/null

getent hosts
```

```bash
ldapsearch -x
```

#### Alternative
```bash
ldapsearch -x -H ldap://192.168.56.11 -b "DC=Efrei,DC=fr" 

```

### Verify config
```bash
sudo slapcat
```
### OU creation

#### Create OU Schema
```bash
nano org_unit.ldif
```

```bash
dn: ou=users,dc=Efrei,dc=fr
objectClass: organizationalUnit

dn: ou=groups,dc=Efrei,dc=fr
objectClass: organizationalUnit
```

```bash
ldapadd -W -D "cn=admin,dc=Efrei,dc=fr" -x -f org_unit.ldif
```

#### Create groups

```bash
dn: cn=teachers, ou=groups, dc=Efrei,dc=fr
objectClass: posixGroup
objectClass: top
gidNumber: 6001
cn: teachers

dn: cn=students, ou=groups, dc=Efrei, dc=fr
objectClass: posixGroup
objectClass: top
gidNumber: 6002
cn: students
```


#### Create users
```bash
nano souhei.ldif
```

```bash

dn: uid=pierre.dupont, ou=users, dc=Efrei, dc=fr
objectClass: person
objectClass: top
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: posixAccount
uidNumber: 2
gidNumber: 6002
homeDirectory: /home/pierre
loginShell: /bin/bash
uid: pierre.dupont
sn: dupont
cn: pierre dupont
mail: pierre.dupont@efrei.fr
userPassword: pierre
```

```bash
dn: uid=souheib.yousfi, ou=users,dc=Efrei,dc=fr
objectClass: person
objectClass: top
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: posixAccount
uidNumber: 1
gidNumber: 6001
homeDirectory: /home/souheib
loginShell: /bin/bash
uid: souheib.yousfi
sn: yousfi
cn: souheib yousfi
mail: souheib.yousfi@efrei.fr
userPassword: souheib
```

```bash
# Add the user to the students group
dn: cn=students,ou=Groups,dc=example,dc=com
changetype: modify
add: member
member: uid=jsmith,ou=People,dc=example,dc=com
```


```bash
 ldapadd -W -D "cn=admin,dc=Efrei,dc=fr" -x -f pierre.ldif
```


### Modify 

```bash
ldapmodify -W -D 'cn=admin,dc=Efrei, dc=fr' -f modifier_pierre -v
```

```bash
dn: uid=pierre.dupont, ou=users, dc=Efrei, dc=fr
changetype: modify
add: mobile
mobile: (0033) 65656565
--
replace: mobile
mobile: (00216) 65757565
--
delete: mail
```


### Config LDAPS

#### Request a certificate and set permissions

```bash
if [ ! -d "/etc/ldap/ssl" ]; then sudo mkdir /etc/ldap/ssl ; fi
cd /etc/ldap/ssl
sudo openssl genpkey -algorithm RSA -out server-key.pem
sudo openssl req -new -key server-key.pem -out server-csr.pem
sudo openssl x509 -req -in server-csr.pem -signkey server-key.pem -out server-cert.pem
sudo chown openldap:openldap server-cert.pem
sudo chown openldap:openldap server-key.pem
sudo chmod 400 server-key.pem
```


#### 
```bash
nano cert.dif

dn: cn=config
changetype: modify
replace: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/ldap/ssl/server-cert.pem
-
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/ldap/ssl/server-cert.pem
-
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/ldap/ssl/server-key.pem
-
replace: olcTLSVerifyClient
olcTLSVerifyClient: never
```

#### Set config

```bash
sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f cert.ldif
```

#### Enable protocols and restart service

```bash
nano /etc/default/slapd 

SLAPD_SERVICES="ldap:/// ldapi:/// ldaps:///"
```

```bash
sudo service slapd restart
```

#### Check with user auth

```bash
ldapwhoami -x -D "uid=pierre.dupont, ou=users, dc=Efrei, dc=fr" -W
```

### LDAPS on client

#### Verify connection to server

```bash
openssl s_client -showcerts -connect Efrei.fr:636 | head -n 2
```

#### Get cerificate on client from server via scp

```bash
#1. in server machine copy it to a folder 
 sudo cp /etc/ldap/ssl/server-cert.pem .
#2. transfer it to client via scp

#3. Modify client file `/etc/ldap/ldap.conf`

BASE dc=Efrei,dc=fr
URI ldaps://Efrei.fr:636

TLS_CACERT /home/pierre/Desktop/cert/server-cert.pem
```

#### Test

```bash
ldapsearch -x -H ldaps://Efrei.fr  "(sn=dupont)" -LLL  -d 1
```


## Evaluation 1 

### Install  LDAP Account Manager and deps

```bash 
sudo apt install apache2 php php-cgi libapache2-mod-php php-mbstring php-common php-pear ldap-account-manager -y

sudo a2enconf php*-cgi
systemctl start apache2
systemctl reload apache2
```

### Visit LDAP Manager on localhost

```bash
curl http://localhost/lam
```

## Evaluation 2:  Enable SSH accès

```bash
cat /etc/nsswitch.conf 
# /etc/nsswitch.conf
#
# Example configuration of GNU Name Service Switch functionality.
# If you have the `glibc-doc-reference' and `info' packages installed, try:
# `info libc "Name Service Switch"' for information about this file.

passwd:         ldap files systemd
group:          ldap files systemd
shadow:         ldap files
gshadow:        files
```

```bash
sudo cat /etc/nslcd.conf
# /etc/nslcd.conf
# nslcd configuration file. See nslcd.conf(5)
# for details.

# The user and group nslcd should run as.
uid nslcd
gid nslcd

# The location at which the LDAP server(s) should be reachable.
uri ldaps://Efrei.fr:636

# The search base that will be used for all queries.
base dc=Efrei,dc=fr

# The LDAP protocol version to use.
#ldap_version 3

# The DN to bind with for normal lookups.
#binddn cn=annonymous,dc=example,dc=net
#bindpw secret

# The DN used for password modifications by root.
#rootpwmoddn cn=admin,dc=example,dc=com

# SSL options
#ssl off
tls_reqcert never
tls_cacertfile /etc/ssl/certs/ca-certificates.crt

# The search scope.
#scope sub
```

## TIPS
[From Openlap installtion to ldaps domain creation with user session activated](https://mittaltarun9715.medium.com/how-to-setup-openldap-server-and-client-installation-in-ubuntu-18-04-with-password-caching-d508a9e80642)