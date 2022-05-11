# LDAP_Server
Tutoriel d'installation et de configuration d'un serveur LDAP sous distributions basées sur Debian

## Liens utiles:
Pour distro basé sur debian: https://devconnected.com/how-to-setup-openldap-server-on-debian-10/

Pour CentOS (obsolete): http://golinuxcloud.com/install-and-configure-openldap-centos-7-linux

Doc officielle d'OpenLDAP: https://openldap.org/doc/admin24/index.html

## Listing de commandes (en root):

1. Installation des paquets:
    `apt install install slapd ldap-utils libnss-ldap`
2. Choix du mot de passe admin

3. Configuration assistée du service LDAP (via le démon nommé `slap`)
    
    3.1 Lancement de l'assistant: `dpkg-reconfigure slapd`
    
    3.2 Fournir le distinguished name (DN) souhaité (ici `dc=anorlondo,dc=org` )
    
    3.3 Fournir le nom de l'organisation souhaité (ici pareil que le DN: `dc=anorlondo,dc=org` )
    
    3.4 Fournir le mot de passe admin souhaité (ici `admin` )
    
    3.5 Choisir la base de donnée souhaitée (ici on laisse MariaDB )
    
3. Pour vérifier que tout s'est bien passé:
    `slapcat`

3bis. Il est aussi possible d'utiliser `slapcat -b cn=config` 
    
4. Emplacement à regarder:
    
    4.1 `/etc/ldap` pour la configuration ldap, les schémas et autres
    
    4.2 `/var/lib/ldap` pour les bases de données

5. Recherche simple pour voir la base de donnée actuelle
    `ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config olcDatabase=\*`
    
6. Pour tester la configuration: il faut vérifier que tout correspond en répétant l'étape 3 et lancer:
    `slaptest -u`
    
7. Ajouter si besoin les schémas de base `cosine`, `nis` et `inetorgperson`
    
    7.1 `ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif`
    
    7.2 `ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif`
    
    7.3 `ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif`
    
8. Ajouter la base de donnée `data.ldif` (disponible dans ce repo) (le mot de passe du compte admin LDAP sera demandé)
    `ldapadd -f data.ldif -D cn=admin,dc=anorlondo,dc=org -W`

14bis. Si erreur, il faut relancer le service LDAP (si ça ne marche toujours pas, il faut la configuration et relancer la commande de l'étape 14 en ajoutant l'option -c)
    `systemctl restart slapd`
    
9. À ce stade on doit pouvoir faire des recherches sur son propre serveur LDAP avec `ldapsearch` ! Félicitation. Pour accéder à votre serveur depuis un client extérieur, il faut penser à demander au pare-feu d'ouvrir les ports utilisés par le service d'annuaire:
    
    9.1 Avec `firewall-cmd`
    
        9.1.1 `firewall-cmd --add-service=ldap --permanent`

        9.1.2 `firewall-cmd --add-port=389/tcp --permanent`

        9.1.3 `firewall-cmd --reload`
    
    9.2. Si le parefeu est `ufw` c'est la commande suivante qu'il faut lancer: `ufw allow 389` (on peut vérifier avec `ufw status`)


10. Ensuite on peut configurer un client LDAP (cela peut être une application dédiée de carnet d'adresse comme KAddressBook sous KDE, ou cela peut être intégré dans un module dans un client de mail comme `thunderbird`)
