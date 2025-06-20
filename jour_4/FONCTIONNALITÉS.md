
#  Documentation — Jour 1

## Jour 1
Jeudi 19 juin 2025

##  Étudiant
Nom : Omar Talibi  
Machine : VM Debian  
Utilisateur : projet

---

### 1. Authentification
- Système d'inscription (`register`) et de connexion (`login`)
- Stockage sécurisé avec base SQLite (table `users`)
- Gestion des erreurs (utilisateur déjà existant, mot de passe incorrect)

### 2. Système de canaux
- Prise en charge de plusieurs canaux (ex : #général, #dev, #admin)
- L'utilisateur envoie des messages dans un canal spécifique
- Diffusion uniquement aux utilisateurs connectés dans le bon canal

### 3. Stockage persistant
- Utilisation de SQLite (`classcord.db`) pour les utilisateurs et les messages
- Table `users` avec `id`, `username`, `password`
- Table `messages` avec `id`, `from_user`, `content`, `timestamp`, `channel`

### 4. Export des messages
- Script `export_messages.py` permettant l’export en JSON ou CSV

### 5. Messages système personnalisés
- Ajout d’une fonction `send_system_message()`
- Notifications de connexion, déconnexion, alertes serveur

### 6. Logging enrichi
- Fichier `server.log` détaillant :
  - Heure de l’événement
  - Type d’événement
  - Utilisateur concerné (le cas échéant)
  - Erreurs ou anomalies détectées

### 7. Client Classcord
- Interface ligne de commande
- Envoi de messages sous forme JSON
- Affichage clair de la connexion, erreur ou retour serveur

## Fichiers inclus
- `server_classcord.py` : serveur principal
- `client_classcord.py` : client ligne de commande
- `db.py` : gestion base SQLite
- `export_messages.py` : export des messages
- `classcord.db` : base SQLite générée automatiquement
- `FONCTIONNALITÉS.md` : ce fichier
- `README.md` : guide d’utilisation

## Captures d'écrans
![alt text](<Capture d'écran 2025-06-19 170555-1.png>)
![alt text](<Capture d'écran 2025-06-19 162423.png>)
![alt text](<Capture d'écran 2025-06-19 142307.png>)