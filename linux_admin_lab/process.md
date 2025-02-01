# LAB 1

- Copy m1.tar in nfs_share
- connect to m1 and get m1.tar via nfs
- copy it to m1 folder then untar it and copy the `student` folder to /home/
- Populate groud, passwd and shadow file to their respective folder
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

# Tips

```bash
sudo cat shadow | tail -6
sudo cat group | tail -2
sudo cat passwd | tail -6
```