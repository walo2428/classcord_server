#  Documentation — Jour 5

## Jour 5
Vendredi 20 juin 2025

##  Étudiant
Nom : Omar Talibi  
Machine : VM Debian  
Utilisateur : projet


### Cloner le projet

```bash
git clone https://github.com/walo2428/classcord-server.git
cd classcord-server
```

## Lancer le serveur (manuel)

```bash
python3 server_classcord.py
```

## Pourquoi Docker ?
- Portabilité et reproductibilité de l’environnement serveur
- Isolation des dépendances et du processus de lancement
- Déploiement simplifié avec `docker run` ou `docker-compose`

## Dockerfile
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . /app
RUN pip install --no-cache-dir -r requirements.txt || true
EXPOSE 12345
CMD ["python", "server_classcord.py"]
```

## docker-compose.yml
```yaml
version: '3'
services:
  classcord:
    build: .
    ports:
      - "12345:12345"
    restart: unless-stopped
```

## Commandes utiles
```bash
docker build -t classcord-server .
docker run -it --rm -p 12345:12345 classcord-server
```
# Documentation de Connexion

## Adresse et port du serveur
- **IP** : 10.0.108.42 (adresse de l’hôte physique, NAT activé)
- **Port** : 12345

## Schéma réseau simplifié
```text
Poste SLAM ──(NAT)──▶ Poste SISR ──▶ VM Linux ──▶ Port 12345
```

## Exemple d'utilisation client
1. Saisir l’IP `10.0.108.42` et le port `12345` dans le client Java
2. Se connecter en tant qu'invité ou utilisateur enregistré
3. Envoyer un message pour vérifier la communication

## Logs de connexion réussie
```
[INFO] 2025-06-17 14:23:05 - Connexion de user1 depuis 10.0.108.34
[INFO] 2025-06-17 14:23:10 - Message reçu : "Bonjour !"
```
# Fonctionnalités personnalisées du serveur

## Canaux de discussion
- Implémentation de canaux (#général, #dev, #admin)
- Aiguillage des messages en fonction du canal sélectionné

## Persistance des données
- Utilisation de SQLite pour stocker les utilisateurs et messages avec horodatage
- Export possible au format CSV

## Messages système personnalisés
- Message d’arrivée, départ, alerte serveur avec `send_system_message()`

## Console d’administration (optionnelle)
- Interface CLI pour surveiller les utilisateurs, envoyer des messages système, etc.

## Exemple de message JSON attendu
```json
{
  "type": "message",
  "subtype": "global",
  "channel": "#dev",
  "from": "bob",
  "content": "Ping pour les devs !"
}
```
# ClassCord Server - Documentation Technique

## Projet BTS SIO Semaine Intensive 2024
Ce dépôt contient l'ensemble des fichiers techniques et de documentation relatifs au déploiement, à la sécurisation et à la gestion du serveur ClassCord développé dans le cadre du projet BTS SIO SISR.

## Contenu
- `README.md` – Présentation générale du projet
- `DOC_CONNEXION.md` – Instructions de connexion pour les clients SLAM
- `SECURITE.md` – Mécanismes de sécurité mis en œuvre
- `FONCTIONNALITES.md` – Fonctionnalités ajoutées et personnalisations
- `CONTAINERS.md` – Détail de la containerisation Docker

# Sécurité du serveur ClassCord

## Journalisation (logging)
- Logs écrits dans `/var/log/classcord.log`
- Utilisation du module `logging` de Python avec horodatage

## Protection contre les attaques (fail2ban)
- Installation de fail2ban
- Configuration de règles pour bloquer les IP après connexions anormales répétées

## Pare-feu
- UFW configuré pour n'autoriser que le port `12345/tcp`
- Accès restreint à l’adresse locale uniquement si nécessaire

## Sauvegarde automatique
- Script cron configuré pour sauvegarder `users.pkl` toutes les heures :
```bash
0 * * * * cp /home/classcord/classcord-server/users.pkl /home/classcord/backups/users-$(date +\%F-\%H\%M).pkl
```
