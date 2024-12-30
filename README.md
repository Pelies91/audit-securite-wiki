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


   - Eléments relevés :
     - Réseau de la salle : `172.16.110.0/24`.
     - Nombre de PCs : `38`.
     - Nombre de Switchs : `12`.

### **2.2. Découverte Automatisée avec Nmap**

#### **Commandes exécutées :**
1. **Ping sweep pour détecter les hôtes actifs :**
   ```bash
   sudo nmap -sn 192.168.1.0/24
   ```
   - Résultat attendu : Liste des adresses IP actives.

2. **Détection des services et versions :**
   ```bash
   sudo nmap -sV -A 192.168.1.10
   ```
   - Exemple de sortie :
     ```
     PORT     STATE SERVICE     VERSION
     22/tcp   open  ssh         OpenSSH 7.4 (protocol 2.0)
     80/tcp   open  http        Apache httpd 2.4.6
     ```

---

## **3. Utilisation d’Outils d’Analyse**

### **3.1. Nmap**

1. **Scan complet des ports sur un hôte :**
   ```bash
   sudo nmap -p- 192.168.1.10
   ```
2. **Scan de vulnérabilités spécifiques :**
   ```bash
   sudo nmap --script vuln 192.168.1.10
   ```

### **3.2. Masscan**

1. **Scan rapide d’un réseau complet :**
   ```bash
   sudo masscan -p80,443 192.168.1.0/24 --rate=1000
   ```

### **3.3. OpenVAS**

1. **Installation et configuration :**
   ```bash
   sudo apt install openvas
   sudo gvm-setup
   ```
2. **Analyse via l’interface graphique :**
   - URL : `https://<IP>:9392`.
   - Créez un scan pour votre plage IP.

### **3.4. Nikto**

1. **Analyse d'une application web :**
   ```bash
   nikto -h http://192.168.1.10
   ```
   - Exemple de sortie :
     ```
     + Server: Apache/2.4.6 (Unix)
     + The X-XSS-Protection header is not set.
     + The server allows TRACE requests, which could cause security issues.
     ```

---

## **4. Tests d’Intrusion**

### **4.1. Utilisation de Metasploit**

1. **Lancer Metasploit :**
   ```bash
   msfconsole
   ```
2. **Recherche de modules :**
   ```bash
   search ssh
   ```
3. **Configurer l’exploit :**
   ```bash
   use exploit/unix/ssh/openssh
   set RHOST 192.168.1.10
   set RPORT 22
   run
   ```

---

## **5. Recommandations**

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

