
#  Sécurité du projet Classcord

## Jour 3
Mercredi 18 juin 2025

##  Étudiant
Nom : Omar Talibi  
Machine : VM Debian  
Utilisateur : projet

##  Méthodes de journalisation

- Les journaux du serveur sont stockés dans le fichier :
  ```
  /var/log/classcord/classcord.log
  ```
- Exemple de log généré par le serveur :
  ```
  2025-06-18 14:58:12,398 - INFO - Utilisateurs chargés : []
  2025-06-18 14:58:12,398 - INFO - Serveur en écoute sur 0.0.0.0:12345
  ```

##  Fail2Ban

- Fail2Ban a été installé avec :
  ```
  sudo apt install fail2ban
  ```
- Fichier de configuration :
  ```
  /etc/fail2ban/jail.local
  ```
- Exemple de section ajoutée :
  ```ini
  [classcord]
  enabled = true
  port    = 12345
  filter  = classcord
  logpath = /var/log/classcord/classcord.log
  maxretry = 3
  bantime = 600
  ```

- Filtre personnalisé dans `/etc/fail2ban/filter.d/classcord.conf` :
  ```ini
  [Definition]
  failregex = .*Tentative.*depuis <HOST>
  ignoreregex =
  ```

##  Exemple d’attaque bloquée

- Extrait du log (tentative d’accès illégitime) :
  ```
  [ERREUR] Problème avec ('10.0.108.45', 54321): Tentative de connexion non autorisée
  ```

- Vérification avec Fail2Ban :
  ```
  sudo fail2ban-client status classcord
  ```
  Résultat :
  ```
  Status for the jail: classcord
  Banned IP list: 10.0.108.45
  ```

##  Sauvegarde automatique

- Script de sauvegarde : `sauvegarde.sh`
  - Copie le fichier `users.pkl` dans le dossier `backups/` avec un timestamp.

- Exemple :
  ```
  /home/projet/BTS_SIO/classcord-server/backups/users_20250618_125001.pkl
  ```

- Tâche `cron` active (vérifiée avec `crontab -l`) :
  ```
  0 * * * * cp /home/classcord/classcord-server/users.pkl /home/classcord/backups/users-$(date +\%F-\%H\%M).pkl
  ```

  captures d'écrans:<<<>>>
![alt text](<Capture d'écran 2025-06-18 110058.png>)
![alt text](<Capture d'écran 2025-06-18 110234.png>)
![alt text](<Capture d'écran 2025-06-18 122527.png>)
![alt text](<Capture d'écran 2025-06-18 125524.png>)
