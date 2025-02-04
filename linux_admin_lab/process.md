# LAB 1

- Copy m1.tar in vbox_share
- connect to m1 and get m1.tar via nfs
- copy it to m1 folder then untar it and copy the `student` folder to /home/
- Populate groud, passwd and shadow file to their respective folder

# Tips

```bash

cp home/student/ /home/

cd /home/student

```
```bash
sudo cat shadow | tail -6
sudo cat group | tail -2
sudo cat passwd | tail -6
```

```bash
sudo cat group | tail -2 | sudo tee -a /etc/group
sudo cat shadow | tail -6 | sudo tee -a /etc/shadow
sudo cat passwd | tail -6 | sudo tee -a /etc/passwd
```

## Questions (m1)

>    Pour chaque utilisateur ayant un uid supérieur à 999, déterminez :
> -  Le nom de connexion.
> -  Le numéro d'utilisateur uid.
> -  Le numéro du groupe principal gid.
> -  Le ou les numéro(s) du groupe secondaire.
> -  Le nom complet, numéro de bureau; numéro de téléphone travail, numéro de téléphone personne.
> -  Le répertoire personnel (c'est également le répertoire de connexion).
> -  L'interpréteur de commande.
> -  Le mot de passe.
> -  Pour chaque utilisateur réalisez un test de connexion sur l'équipement m1. Dans le cas où l'accès serait interdit à
l'utilisateur, déterminez les causes de cette interdiction sans les corriger


---

# Réponse (m1)
```bash
awk -F: '$3 > 999 {print $1 ":" $3 ":" $4 ":" $5 ":" $6 ":" $7}' /etc/passwd | while IFS=: read user uid gid info home shell; do 
    secondary_groups=$(id -G "$user" | tr ' ' '\n' | grep -v "^$gid$" | while read gid; do getent group "$gid" | awk -F: '{print $1" (id:"$3")"}'; done | paste -sd ", " -) 
    password_status=$(sudo grep "^$user:" /etc/shadow | cut -d: -f2) 
    echo "Utilisateur: $user"
    echo "UID: $uid"
    echo "GID: $gid"
    echo "Groupes secondaires: $secondary_groups"
    echo "Infos: $info"
    echo "Répertoire personnel: $home"
    echo "Interpréteur de commande: $shell"
    echo "Mot de passe: $password_status"
    echo "--------------------------------------"
done
```


# Réponse (m2)

```bash
#!/bin/bash

# Création des groupes
groupadd -g 1600 administration
groupadd -g 1700 recherche
groupadd -g 1800 production

# Création des répertoires principaux
mkdir -p /home/xl_networks/{administration,recherche,production}
mkdir /home/users/{zetaufrait,terrueur,bon,assin,eparbal,menvussa,liguli}

# Attribution des permissions aux répertoires principaux
chown root:administration /home/xl_networks/administration
chown root:recherche /home/xl_networks/recherche
chown root:production /home/xl_networks/production
chmod 770 /home/xl_networks/{administration,recherche,production}

# Création des utilisateurs
useradd -m -d /home/users/zetaufrait -u 1601 -g administration -c "ZETAUFRAIT Melanie, Bureau: 1601, Tel: 01 69 33 61 00" -s /bin/bash mzetaufrait
echo "melanie" | passwd --stdin mzetaufrait
chmod 700 /home/users/zetaufrait
setfacl -m u:mzetaufrait:rx /home/xl_networks/administration
setfacl -m u:mzetaufrait:r /home/xl_networks/production

useradd -m -d /home/users/terrieur -u 1602 -g administration -c "TERRIEUR Alex, Bureau: 1602, Tel: 01 69 33 61 00" -s /bin/bash aterrieur
echo "alex" | passwd --stdin aterrieur
chmod 700 /home/users/terrieur
setfacl -m u:aterrieur:rx /home/xl_networks/administration
setfacl -m u:aterrieur:r /home/xl_networks/production

useradd -m -d /home/users/bon -u 1603 -g administration -c "BON Jean, Bureau: 1603, Tel: 01 69 33 61 00" -s /bin/bash jbon
echo "jean" | passwd --stdin jbon
chmod 700 /home/users/bon
setfacl -m u:jbon:rwx /home/xl_networks/administration
setfacl -m u:jbon:r /home/xl_networks/production

useradd -m -d /home/users/assin -u 1701 -g recherche -c "ASSIN Marc, Bureau: 1701, Tel: 01 69 33 61 00" -s /bin/bash massin
echo "marc" | passwd --stdin massin
chmod 700 /home/users/assin
setfacl -m u:massin:rwx /home/xl_networks/recherche
setfacl -m u:massin:r /home/xl_networks/production

useradd -m -d /home/users/eparbal -u 1702 -g recherche -c "EPARBAL Gilles, Bureau: 1702, Tel: 01 69 33 61 00" -s /bin/bash geparbal
echo "gilles" | passwd --stdin geparbal
chmod 700 /home/users/eparbal
setfacl -m u:geparbal:rwx /home/xl_networks/recherche
setfacl -m u:geparbal:r /home/xl_networks/production

useradd -m -d /home/users/menvussa -u 1801 -g production -c "MENVUSSA Gerard, Bureau: 1801, Tel: 01 69 33 61 00" -s /bin/bash gmenvussa
echo "gerard" | passwd --stdin gmenvussa
chmod 700 /home/users/menvussa
setfacl -m u:gmenvussa:rwx /home/xl_networks/production

useradd -m -d /home/users/liguili -u 1802 -g production -c "LIGUILI Guy, Bureau: 1802, Tel: 01 69 33 61 00" -s /bin/bash gliguili
echo "guy" | passwd --stdin gliguili
chmod 700 /home/users/liguili
setfacl -m u:gliguili:rwx /home/xl_networks/production
```