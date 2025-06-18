
#  Documentation — Jour 1

## Jour 2
Mardi 17 juin 2025

##  Étudiant
Nom : Omar Talibi  
Machine : VM Debian  
Utilisateur : projet

---

## Objectifs de la journée

## Étapes réalisées

### 1. Clonage et lancement local
```bash
git clone https://github.com/AstrowareConception/classcord-server.git
cd classcord-server
python3 server_classcord.py
```
Serveur accessible localement sur `localhost:12345`.

### 2. Vérification du port
```bash
sudo ufw allow 12345/tcp
ss -tulpn | grep 12345
```
Port bien ouvert pour écoute TCP.

### 3. Accès depuis une machine SLAM (via IP publique + redirection NAT)
- IP publique hôte : `10.0.108.XX`
- Port exposé : `12345`
- Connexion client réussie : ✅

### 4. Utilisateur système dédié
```bash
sudo useradd -m classcord
sudo passwd classcord
su - classcord
```

### 5. Service `systemd`
Fichier `/etc/systemd/system/classcord.service` :
```ini
[Unit]
Description=Serveur ClassCord
After=network.target

[Service]
User=classcord
WorkingDirectory=/home/classcord/classcord-server
ExecStart=/usr/bin/python3 /home/classcord/classcord-server/server_classcord.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Activation :
```bash
sudo systemctl daemon-reexec
sudo systemctl enable --now classcord.service
```

### 6. Dockerisation
- Fichier `Dockerfile` créé à la racine du projet :
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . /app
RUN pip install --no-cache-dir -r requirements.txt || true
EXPOSE 12345
CMD ["python", "server_classcord.py"]
```

- Construction et exécution :
```bash
docker build -t classcord-server .
docker run -it --rm -p 12345:12345 classcord-server
```

### 7. Documentation technique créée
- `DOC_CONNEXION.md` : Instructions de connexion depuis un poste client SLAM
- `README.md` : État du serveur et configuration réseau

##  Captures d’écran (non incluses ici)
![alt text](<Capture d’écran 2025-06-18 091940.png>)
![alt text](<Capture d'écran 2025-06-17 113419.png>)
- `systemctl status classcord.service`

##  Compétences mobilisées
- Rendre un service accessible sur un réseau pédagogique restreint
- Lancer un service automatiquement au démarrage
- Créer une image Docker prête à être redéployée
- Rédiger une documentation technique claire

---
 Dernière mise à jour : 2025-06-18
