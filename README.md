# audit-securite-wiki
# Wiki d'Audit de Sécurité

Bienvenue sur le wiki d'audit de sécurité. Ce document présente un guide complet pour effectuer un audit de sécurité d'une infrastructure réseau.

---

## **1. Objectifs de l'Audit**

L’objectif de ce TP n°2 est d’appliquer les notions théoriques et pratiques des séances de CM et de TD. De cette façon, nous allons pouvoir comprendre l’infrastructure du réseau cible en documentant sa topologie. Nous pourrons également identifier les vulnérabilités et les faiblesses à l’aide de divers outils d’analyse tout en élaborant une checklist pour structure un audit de sécurité. Enfin, nous prendrons soin d’exécuter des tets d’intrusion en simulant des attaques réelles et en proposant des recommandations pratiques pour sécuriser le réseau.

L’organisation de ce TP sera ici divisée en 4 parties à savoir :
-	L’Analyse Fonctionnelle Simulée
-	L’utilisation d’Outils d’Analyse
-	La Préparation de l’Audit
-	Les tests d’intrusion

Une conclusion sera ensuite rédigée avec le retour des observations et expériences sur ce TP, en portant attention sur l'important de ces exercices dans le contexte des audits de sécurité.


---

## **2. Analyse Fonctionnelle Simulée**


Ce premier exercice va nous permettre de comprendre la topologie réseau et ses points faibles potentiels. La première étape reste donc d'identifier les différents équipements réseaux de notre topologie comme les PC, les switch, les serveurs webs etc. Pour ce faire voici une maquette draw.io qui récapitule tous ces événements :

![Maquette de topologie réseau](topologie.jpg)


   - Eléments relevés :
     - Réseau de la salle : `172.16.110.0/24`.
     - Nombre de PCs : `38`.
     - Nombre de Switchs : `12`.

Les différentes commandes à suivre seront donc effectuées depuis mon ordinateur portable, ce dernier étant directement au réseau de la salle de l'IUT via un cable ethernet. De plus, on aurait également pu essayer de construire et de découvrir la topologie du réseau de facon automatisée avec l'utilisation des 2 commandes Nmap : 

**Ping sweep pour détecter les hôtes actifs :**
   ```bash
   sudo nmap -sn 192.168.1.0/24
   ```

**Détection des services et versions :**
   ```bash
   sudo nmap -sV -A 192.168.1.10
   ```

Pour finir, on peut conclure que la topologie d'un réseau peut fortement influencer la sécurité. En effet, une segmentation réseau bien concue (séparation LAN/DMZ...) limite la propagation des attaques tandis qu'une mauvaise configuration expose des ressources sensibles.

---

## **3. Utilisation d’Outils d’Analyse**

Afin de réaliser cette partie, nous allons nous servir d'outils tels que nmap et masscan depuis une VM Kali Linux dans le but d'analyser les résultats de ces commandes pour identifier les vulnérabilités. Pour masscan, on utilisera cette commande qui permettra de détecter les ports ouverts sur des machines cibles de sorte à déceler les vulnérabilités potentielles à partir du résultat. Ce dernier est concu pour effectuer des scans réseau à une vitesse élevée, en analysant des millions d'adresses IP en un temps réduit. En termes d'avatanges, on peut considérer que masscan est bien plus rapide que nmap pour le balayage des ports sur de grandes plages IP. Cependant il fournit moins de détails et il est souvent utilisé en compléments d'outils. Pour finir sa vitesse de balayage est hautement personnalisable car elle dépend des techniques d'envoi de paquets TCP/UDP en masse, de l'ordre de millions de paquets par seconde maximum.

En ce qui concerne nmap, cet outil permet d'analyser en profondeur les réeaux et systèmes, de détecter des ports ouverts, d'identifier des services actifs, et de déterminer les versions des logiciels ou systèmes d'exploitation. On l'utilisera ici pour compléter l'analyse en profondeur des services exposés. On se servira également de l'outil nikto pour scanner les applications web sur les hôtes détectés.


### **Nmap**

1. **Scan complet des ports depuis une adresse de réseau :**
   ```bash
   sudo nmap -sV -O 172.16.110.0/24 -oN scan_resultats3.txt
   ```

   ![Résultat de la commande nmap](commande_nmap.png)

La commande effectuée réalise un scan réseau sur le sous-réseau 172.16.110.0/24 pour détecter les ports ouverts, identifier les versions des services associés ainsi qu'analyser le système d'exploitation des hôtes. Cette commande possède également plusieurs arguments qui sont :

* -sV : Active la détection de versions des services sur les ports ouverts. Cela va permettre d'identifier les logiciels et leurs versions qui seront exécutes depuis les hôtes.
* -O : Active la fonctionnalité de détection du systèmes d'exploitations en analysant les caractéristiques des paquets envoyés et répondus par les hôtes.
* -On scan_resultats3.txt : Enregiste les résultats du scan dans un fichier au format txt.

En examinant de plus près le fichier texte [scan_resultats3.txt](scan_resultats3.txt) qui contient les résultats de la commande nmap exécutée, on parvient à distinguer plusieurs hôtes au sein du résau qui sont :

* 172.16.110.10 : On remarque que l'hôte de nom "xivo-UC-rt100.root" est actif au sein du réseau avec une latence de 0,00091s. Les ports ouverts sur ce dernier sont mutiples et on retrouve :
     - Le port 22/tcp qui représente le port rattaché au service OpenSSH version 8.4 utilisé pour les connexions sécurisées a distance. 
     - Le port 25/tcp qui est utilisé pour un service Postfix SMTPD dans l'optique d'envoie de courriels.
     - Le port 80/tcp qui est relié au serveur web Nginx et le port 443/tcp utilisé pour des connexions chiffrés avec SSL/TLS avec le serveur Nginx (son port alternatif est également mentionné, ce dernier etant le 8443/tcp)
     - Le port 5003/tcp rattaché à FileMaker.
     - Le port 5432/tcp qui est lié à la base de donnée PostgreSQL.
     - Le port 9090/tcp qui est un service inconnu et qui pourrait être un panneau d'administration.
     - Le port 9100/tcp qui correspond à un service lié aux imprimantes réseau (JetDirect).

On peut donc en déduire que la machine ciblée fonctionne sous Linux au vu des services installés.

* 172.16.110.53 : On retrouve également une autre machine sous cette commande, qui possède une latence de 0,0013s et qui héberge un service OpenSSH version 9.2 du a l'utilisation du port 22/tcp.

Cela revient donc à conclure que les résultats montrent que le réseau contient plusieurs hôtes éxecutant des services. On remarque également que certains services peuvent avoir des répercussions sur la latence d'un hôte, en particulier si ce dernier en héberge beaucoup. Les services non identifiées peuvent également nécessister des vérifications afin d'évaluer leurs potentiels risque de sécurité.

### **Masscan**

1. **Scan rapide d’un réseau complet :**
   ```bash
   sudo masscan -p22,80,3306,8080,443 172.16.110.0/24 -oL scans_resultats4.txt
   ```

   ![Résultat de la commande masscan](commande_masscan.png)

La commande précédente utilise masscan afin d'effectuer un outil de scan réseau rapide, pour analyser un sous réseau (172.16.110.0) et détecter les ports ouverts sur des machines cibles. Elle comporte égalemeent certains éléments qui sont :

* -p22,80,3306,8080,443 : Spécifie les ports à scanner, 22 pour SSH, 80 pour HTTPS, 3306 pour Mysql, 8080 pour HTTP alternatif, 443 pour HTTPS.
* -oL scans_resultats4.txt : Spécifie que les résultats du scan seront sauvegardés dans un fichier au format txt.

Le fichier texte [scans_resultats4.txt](scans_resultats4.txt) nous renvoie des informations brutes générées par masscan après un scan des ports spécifiés sur la plage d'adresse 172.16.110.1-254. On remarque que la commande parvient à identifier plusieurs hôtes avec des ports ouverts différents qui sont :
* IP : 172.16.110.10
  - Ports ouverts : 22, 80 et 443 
* IP : 172.16.110.7
  - Ports ouverts : 22, 80 et 443 
* IP : 172.16.110.92
  - Ports ouverts : 80, 3306 et 443 
* IP : 172.16.110.95
  - Ports ouverts : 80, 443 et 3306
* IP : 172.16.110.253
  - Ports ouverts : 22 et 80 
* IP : 172.16.110.97
  - Ports ouverts : 80, 3306 et 443 
* IP : 172.16.110.254
  - Ports ouverts : 22 et 80 
* IP : 172.16.110.63
  - Ports ouverts : 22
* IP : 172.16.110.72
  - Ports ouverts : 22 et 80 
* IP : 172.16.110.111
  - Ports ouverts : 22
* IP : 172.16.110.60
  - Ports ouverts : 22
* IP : 172.16.110.183
  - Ports ouverts : 22 
* IP : 172.16.110.179
  - Ports ouverts : 22 et 80
* IP : 172.16.110.54
  - Ports ouverts : 22
* IP : 172.16.110.53
  - Ports ouverts : 22
* IP : 172.16.110.159
  - Ports ouverts : 22
* IP : 172.16.110.183
  - Ports ouverts : 22
* IP : 172.16.110.86
  - Ports ouverts : 22
* IP : 172.16.110.78
  - Ports ouverts : 22
* IP : 172.16.110.93
  - Ports ouverts : 22
* IP : 172.16.110.80
  - Ports ouverts : 22
* IP : 172.16.110.62
  - Ports ouverts : 22 
* IP : 172.16.110.182
  - Ports ouverts : 22

Enfin, si l'on souhaite faire una analyse des risques par rapport à ces données, on peut tout d'abord dire que le port 22 utilisé pour SSH est présent sur presque toutes les machines. Cela peut indiquer un risque si les mots de passe sont faibles ou que les configurations par défaut sont utilisés. Les ports 80 et 443 nous indiquent également que des services web sont accessibles depuis de nombreuses machines. Il faudra par ailleurs s"assurer que les applications soient protégées contre les attaques connues. Pour finir, le port 3306 indique aussi que la base de données peut être aussi rendue accessible. Il faudra donc prendre en compte ce risque et vérifier que les accès soient bien restreints.

### **Nikto**

1. **Analyse d'une application web :**
   ```bash
   nikto -h http://192.168.1.XX
   ```

   ![Résultat de la commande masscan](commande_nikto.png)
   ![Résultat de la commande masscan](commande_nikto2.png)
   
Enfin, la dernière commande utilise l'outil Nikto, un outil d'analyse de sécurité pour les serveurs web, sur une machine cible spécifique. Le paramètre -h spécifie l'hôte cible, tandis que "172.16.110.XX" représente l'adresse IP de la machine cible ou le serveur web est analysé. L'utilité de cette commande est donc d'identifier les vulnérabilités sur le serveur web et d'évaluer la conformité et la sécurité des applications hébergées depuis la machine.

En regardant de près les 2 captures, on constate que la première cible 2 adresses, à savoir 172.16.110.179 et 172.16.110.92. L'un possède un serveur Nginx et on y décèle plusieurs vulnérabilités comme le HTPP X-Frame-Options qui n'est pas configuré et le header X-Content-Type-Options qui n'est lui aussi pas défini (qui aurait permis à l'utilisateur de forxer une exécution avec un type MIME incorrect. De plus on remarque aussi qu'un fichier "wp-config.php" à été détecté, qui est susceptible de contenir des identifiants sensibles. La deuxième adresse IP quant à elle héberge un serveur Apache et un serveur OpenSSL. On constate la aussi que les header sont absents et que l'accès à "/robots.txt" révèle 6 entrées exploitables. On note aussi une présence de failles connues comme CVE-2006-3918 qui correspond à uné vulnérabilité XSS. Les méthodes HTTP qui sont renseignés peuvent aussi rendre le serveur vunérable aux attaques Cross-Site Tracing.
Pour plus de résultats, le deuxième screen affiché cible lui aussi 2 adresses, qui sont 172.16.110.95 et 172.16.110.78. La première présente des similitudes avec des vulnariblités quasi identiques aux l'adresse hébergant le serveur Apache et OpenSSL. La seconde adresse elle héberge un serveur Caddy (serveur web), qui présente aussi des vulnérabilités avec l'absence des headers HTTP.

---

## **4.Préparation à l’Audit**

### Scénario fictif : Audit de sécurité d’une entreprise
L'entreprise **TechSoft Ltd.**, spécialisée dans le développement d'applications web, dispose de quatre serveurs accessibles depuis Internet. Cet audit vise à évaluer leur niveau de protection contre les cyberattaques. Les outils utilisés incluent **Nmap**, **Masscan**, **Nikto**, et des inspections manuelles.


### - Liste de Vérification pour l’Audit de Sécurité

### 1. Analyse de Surface d’Attaque (Reconnaissance)
- **Scan des ports et services :**
  - Identifier les ports ouverts (outils : Nmap, Masscan).
  - Vérifier les services actifs et leurs versions (exemple : Apache, MySQL, etc.).
  - Évaluer si les ports inutilisés peuvent être fermés.
- **Vérification des règles de pare-feu :**
  - Vérifier si les règles de filtrage permettent uniquement le trafic nécessaire.

### 2. Sécurité des Services Web
- **Vérification des vulnérabilités avec Nikto :**
  - Absence des headers HTTP critiques (`X-Frame-Options`, `X-Content-Type-Options`, `Strict-Transport-Security`).
  - Présence de fichiers sensibles (ex. : `/wp-config.php`, `/backup/`).
  - Vérification des chemins dans `/robots.txt`.
- **Validation des certificats SSL :**
  - Vérifier l’expiration et la force des certificats TLS.

### 3. Gestion des Mises à Jour
- **Versions des logiciels :**
  - Identifier les versions obsolètes (Apache, OpenSSL, PHP, etc.).
  - Vérifier les vulnérabilités connues (ex. : CVE-2006-3918 détectée sur Apache).

### 4. Politique de Mots de Passe
- **Sécurité des mots de passe :**
  - Vérifier les exigences en matière de mots de passe (longueur, complexité).
  - Confirmer l’absence de mots de passe par défaut.

### 5. Sécurité Réseau
- **Segmentation réseau :**
  - Vérifier si les serveurs critiques sont isolés des autres services.
  - Confirmer l’utilisation d’une DMZ pour les services publics.
- **Méthodes HTTP autorisées :**
  - Désactiver les méthodes dangereuses (`TRACE`, `PUT`).

### 6. Surveillance et Journalisation
- **Analyse des logs :**
  - Vérifier si les journaux système et applicatifs sont activés.
  - S’assurer que les logs sont protégés contre les modifications non autorisées.

---

# Questions de Discussion

# Quels sont les éléments essentiels à inclure dans une liste de vérification ?
Une liste de vérification efficace doit inclure :
- La surface d'attaque (ports, services).
- La sécurité des applications (versions, vulnérabilités).
- La configuration réseau (pare-feu, segmentation).
- La gestion des accès et des mots de passe.
- Les mises à jour et correctifs de sécurité.
- La surveillance et la réponse aux incidents.

# Comment la liste de vérification peut-elle guider un audit de sécurité ?
- **Priorisation des actions :** Identifie les vulnérabilités critiques à corriger en premier.
- **Structure et cohérence :** Fournit un guide clair pour couvrir tous les aspects de la sécurité.
- **Documentation :** Permet de justifier les recommandations et de fournir des preuves d’analyse.
- **Amélioration continue :** Peut être réutilisée et adaptée pour des audits futurs.

# Outils utilisés
- **Nmap** : Scan des ports et services.
- **Masscan** : Analyse rapide des ports ouverts.
- **Nikto** : Détection des vulnérabilités sur les serveurs web.

---

## **5. Tests d’Intrusion et d’Attaques**

### **5.1. Résumé des vulnérabilités trouvées**

- Ports inutiles ouverts (ex. : 3306 MySQL).
- Absence de headers de sécurité (Nikto).
- CVE critiques détectées (OpenVAS).

### **5.2. Recommandations pratiques**

1. Fermez les ports inutiles.
2. Mettez à jour les versions des services.
3. Appliquez des règles strictes au pare-feu.

---

## **6. Conclusion**

---

