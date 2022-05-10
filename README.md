# LDAP_Server
Tutoriel d'installation et de configuration d'un serveur LDAP sous CentOS 7

## Liens utiles:
Pour CentOS (obsolete): http://golinuxcloud.com/install-and-configure-openldap-centos-7-linux
Pour distro basé sur debian: https://devconnected.com/how-to-setup-openldap-server-on-debian-10/
Doc officielle d'OpenLDAP: https://openldap.org/doc/admin24/index.html

## Listing de commandes (en root):

1. Installation des paquets:
    `yum install openldap openldap-servers openldap-clients -y`
2. Lancement du service LDAP (via le démon nommé `slap`) 
    `systemctl start slapd`
3. Choix du mot de passe
    `slappasswd`
    
/!\ Attention /!\: bien copier le hash donné par la fonction sha. Cela devrait ressembler à un truc du style: {SSHA}+UEFAzQyXhGpYNI+YVgGlyf/XyV4ZmLu 

Pour éviter les erreurs, il est conseillé de copier le résultat dans un fichier (nommé ici `adminpw.hash`):
    `slappasswd >> adminpw.hash`
    
4. Création du fichier `olcSuffix.ldif` (disponible dans ce repo)
5. Création du fichier `olcRootPW.ldif` (disponible dans ce repo, attention à bien remplacer le hash par le sien à l'étape 9 !)
6. Création du fichier `olcAccess.ldif` (disponible dans ce repo)

7. Recherche simple pour voir la base de donnée actuelle
    `ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config olcDatabase=\*`
    
8. Ajout du fichier `olcSuffix.ldif` (cela va remplacer le `olcSuffix` et le `olcRootDN` par défaut pour mettre `dc=anorlondo,dc=org` pour `olcSuffix` et `cn=admin,dc=musicschool,dc=fr` pour `olcRootDN`)
    `ldapmodify -Y EXTERNAL -H ldapi:/// -f olcSuffix.ldif`
    
9. Ajout du fichier `olcRootPW.ldif` (cela va ajouter le hash du mot de passe administrateur dans la base)
    9.1 Le fichier `olcRootPW.ldif` est déjà pré-rempli. Il faut maintenant concaténer le hash:
        `cat adminpw.hash >> olcRootPW.ldif`
    9.2 Il faut supprimer retour à la ligne de sorte à avoir l'attribut `olcRootPW` sur la même ligne que le hash. ça donne un truc comme ça à la dernière ligne du fichier `olcRootPW.ldif`:
        `olcRootPW: {SSHA}+UEFAzQyXhGpYNI+YVgGlyf/XyV4ZmLu`
    9.3 Ajout concret du fichier `olcRootPW.ldif`
        `ldapmodify -Y EXTERNAL -H ldapi:/// -f olcRootPW.ldif`
        
10. Pour vérifier que tout s'est bien passé:
    `ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config olcDatabase=\*`
10bis. Il est aussi possible d'utiliser `slapcat -b cn=config` 

11. Ajout du fichier `olcAccess.ldif` (cela va gérer les droits d'accès du compte admin pour la base de donnée. Source d'erreur "Invalid credentials (49)" si la dernière ligne est mal configurée ou si le hash est mal recopié)
    11.1 Bien vérifier le contenu du fichier. La dernière ligne du fichier doit être:
        `olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read by dn.base="cn=admin,dc=anorlondo,dc=org" read by * none`
    11.2 Ajout concret du fichier `olcAccess.ldif`
        `ldapmodify -Y EXTERNAL -H ldapi:/// -f olcAccess.ldif`
        
12. Pour tester la configuration: il faut vérifier que tout correspond en répétant l'étape 10 et lancer:
    `slaptest -u`
    
13. Ajouter les schémas de base `cosine`, `nis` et `inetorgperson`
    13.1 `ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif`
    13.2 `ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif`
    13.3 `ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif`
    
14. Ajouter la base de donnée `data.ldif` (disponible dans ce repo) (le mot de passe du compte admin LDAP sera demandé)
    `ldapadd -f data.ldif -D cn=admin,dc=anorlondo,dc=org -W`

14bis. Si erreur, il faut relancer le service LDAP (si ça ne marche toujours pas, il faut la configuration et relancer la commande de l'étape 14 en ajoutant l'option -c)
    `systemctl restart slapd`
    
15. À ce stade on doit pouvoir faire des recherches sur son propre serveur LDAP avec `ldapsearch` ! Félicitation. Pour accéder à votre serveur depuis un client extérieur, il faut penser à demander au pare-feu d'ouvrir les ports utilisés par le service d'annuaire:
    15.1 `firewall-cmd --add-service=ldap --permanent`
    15.2 `firewall-cmd --add-port=389/tcp --permanent`
    15.3 `firewall-cmd --reload`

16. Ensuite on peut configurer un client LDAP (cela peut être une application dédiée de carnet d'adresse comme KAddressBook sous KDE, ou cela peut être intégré dans un module dans un client de mail comme `thunderbird`)
